---
title: "[Kubernetes] Calico BGP 완전 정리: Pod 네트워크 라우팅 메커니즘"
date: 2026-02-26 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, calico, bgp, cni, networking, pod-network, route-reflector, bgp-peering]
render_with_liquid: false
---

# Calico BGP (Border Gateway Protocol)

## 1. 개요

**한 줄 요약**: 어디로 가야 목적지에 도달할 수 있는지 알려주는 라우팅 프로토콜

**핵심 개념**:
- 네트워크 간 경로 정보를 교환하는 프로토콜
- 인터넷: AS(Autonomous System) 간 라우팅
- Kubernetes: 노드 간 Pod 네트워크 라우팅

---

## 2. BGP란?

### 2.1. 실생활 비유

```
택배 기사들이 서로 대화하는 것:

택배 기사 A: "나는 강남구 담당이야, 강남 가는 택배는 나한테 줘"
택배 기사 B: "나는 마포구 담당이야, 마포 가는 택배는 나한테 줘"
택배 기사 C: "나는 송파구 담당이야, 송파 가는 택배는 나한테 줘"
    ↓
각자 담당 구역 정보를 서로 공유
    ↓
택배가 올 때 어디로 보낼지 판단
```

### 2.2. 인터넷에서의 BGP

```
KT  AS: "나는 1.1.1.0/24 IP 대역 담당"
SKT AS: "나는 2.2.2.0/24 IP 대역 담당"
LGU+ AS: "나는 3.3.3.0/24 IP 대역 담당"
    ↓
BGP로 서로 경로 광고
    ↓
KT 사용자가 SKT 서버(2.2.2.10) 접속 시
→ "2.2.2.x는 SKT로 보내면 되겠군"
```

**AS (Autonomous System)**: 하나의 관리 주체가 운영하는 네트워크 집합 (예: KT, SKT, Google, AWS 등)

---

## 3. Calico에서의 BGP

### 3.1. 기본 원리

Calico는 BGP를 **Kubernetes 노드 간 Pod 라우팅**에 활용합니다.

```
Node1 calico-node: "나는 10.244.1.0/24 Pod 대역 담당"
Node2 calico-node: "나는 10.244.2.0/24 Pod 대역 담당"
Node3 calico-node: "나는 10.244.3.0/24 Pod 대역 담당"
    ↓
BGP로 서로 광고 (BGP Peering)
    ↓
각 노드의 라우팅 테이블에 등록
    ↓
Pod 간 통신 시 올바른 노드로 라우팅
```

### 3.2. 라우팅 테이블 예시

**Node1의 라우팅 테이블**:

```bash
# ip route
10.244.1.0/24 dev *              # 내 Pod (직접 연결)
10.244.2.0/24 via 192.168.1.11   # Node2 Pod → Node2 IP로 전송
10.244.3.0/24 via 192.168.1.12   # Node3 Pod → Node3 IP로 전송
```

**Node2의 라우팅 테이블**:

```bash
# ip route
10.244.1.0/24 via 192.168.1.10   # Node1 Pod → Node1 IP로 전송
10.244.2.0/24 dev *              # 내 Pod (직접 연결)
10.244.3.0/24 via 192.168.1.12   # Node3 Pod → Node3 IP로 전송
```

### 3.3. Pod 간 통신 흐름

```
Pod A (10.244.1.5, Node1) → Pod B (10.244.2.10, Node2)

1. Pod A에서 패킷 발송
    ↓
2. Node1 라우팅 테이블 확인
   "10.244.2.0/24 → 192.168.1.11 (Node2)"
    ↓
3. Node1 → Node2로 패킷 전송
    ↓
4. Node2에서 수신 → Pod B로 전달
```

---

## 4. BGP Peering

### 4.1. Peering이란?

calico-node끼리 BGP 정보를 주고받는 **연결**을 BGP Peering이라고 합니다.

```
Node1 calico-node ←→ Node2 calico-node  (Peering)
Node1 calico-node ←→ Node3 calico-node  (Peering)
Node2 calico-node ←→ Node3 calico-node  (Peering)
```

### 4.2. Node-to-Node Mesh (기본 방식)

Calico 기본 설정은 **모든 노드가 서로 Peering**하는 방식입니다.

```
노드 3개   → Peering 3개      (3×2÷2)
노드 10개  → Peering 45개     (10×9÷2)
노드 100개 → Peering 4,950개  (100×99÷2)
```

노드 수가 많아질수록 Peering 수가 기하급수적으로 증가하여 BGP 세션 유지 오버헤드가 커집니다. 일반적으로 **50~100개 노드까지 권장**합니다.

### 4.3. Route Reflector 방식 (대규모 클러스터)

100개 이상 노드 환경에서는 중앙 집중화 방식으로 전환합니다.

```
                Route Reflector
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      Node1         Node2         Node3
     (Client)      (Client)      (Client)

Peering:
- Node1 ←→ Route Reflector
- Node2 ←→ Route Reflector
- Node3 ←→ Route Reflector

노드 100개 → Peering 100개만 필요! (O(n²) → O(n))
```

### 4.4. BGP Peering 상태 확인

```bash
kubectl exec -n kube-system calico-node-xxx -c calico-node -- calicoctl node status

# 출력 예시
# IPv4 BGP status
# +--------------+-------------------+-------+----------+-------------+
# | PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
# +--------------+-------------------+-------+----------+-------------+
# | 192.168.1.11 | node-to-node mesh | up    | 10:00:00 | Established |
# | 192.168.1.12 | node-to-node mesh | up    | 10:00:00 | Established |
# +--------------+-------------------+-------+----------+-------------+
```

STATE가 `Established`이면 정상, `start` 또는 `idle`이면 Peering 실패 상태입니다.

---

## 5. calico-node 재시작 시 영향

### 5.1. 재시작 전체 흐름

```
t=0초:  calico-node 재시작 시작
t=1초:  BGP Peering 세션 끊김
t=2초:  다른 노드들이 Node1 경로 정보 삭제
        → 라우팅 테이블에서 "10.244.1.0/24 → 192.168.1.10" 제거
t=3~20초: 다른 노드 → Node1 Pod 통신 불가
t=20초: 새 calico-node Running
t=21초: BGP Peering 재연결
t=22초: "Node1은 10.244.1.0/24 담당" 다시 광고
t=23초: 다른 노드 라우팅 테이블 업데이트
t=25초: 통신 정상화
```

### 5.2. 영향 범위 정리

| 통신 방향 | 영향 | 이유 |
|-----------|------|------|
| **Node1 → 다른 노드 Pod** | ✅ 가능 | Node1의 라우팅 테이블은 유지됨 |
| **다른 노드 → Node1 Pod** | ❌ 불가 (10~30초) | 경로 정보가 삭제됨 |
| **Node1 Pod 간 통신** | ✅ 가능 | 동일 노드 내부 통신 |
| **Service 경유 통신** | ✅ 대부분 가능 | iptables 규칙은 유지됨 |

### 5.3. 재시작 후 확인 명령어

```bash
# 1. BGP Peering 상태 확인
kubectl exec -n kube-system calico-node-xxx -c calico-node -- calicoctl node status

# 2. 라우팅 테이블 확인
kubectl exec -n kube-system calico-node-xxx -c calico-node -- ip route | grep 10.244

# 3. Pod 통신 테스트
kubectl exec -it pod-a -- ping 10.244.2.5

# 4. calico-node 로그에서 BGP 확인
kubectl logs -n kube-system calico-node-xxx -c calico-node | grep -i bgp
```

---

## 6. 트러블슈팅

### 6.1. BGP Peering 실패

**증상**:

```bash
kubectl exec -n kube-system calico-node-xxx -c calico-node -- calicoctl node status
# STATE: start 또는 idle  (Established 아님)
```

**원인 및 해결**:

```bash
# 원인 1: 방화벽에서 BGP 포트(179) 차단
sudo iptables -L -n | grep 179

# 원인 2: calico-node Pod 비정상
kubectl get pods -n kube-system -l k8s-app=calico-node

# 원인 3: 노드 간 통신 불가 → 노드 간 ping 테스트
ping <target-node-ip>

# 해결: calico-node 재시작
kubectl rollout restart daemonset -n kube-system calico-node
```

### 6.2. Pod 간 통신 실패

**증상**:

```bash
kubectl exec -it pod-a -- ping 10.244.2.5
# Network unreachable
```

**확인 순서**:

```bash
# 1. 라우팅 테이블 확인
kubectl exec -n kube-system calico-node-xxx -c calico-node -- ip route | grep 10.244.2.0

# 2. BGP 경로 확인
kubectl exec -n kube-system calico-node-xxx -c calico-node -- calicoctl node status

# 3. 노드 간 직접 ping 테스트
ping <node2-ip>

# 4. NetworkPolicy 확인
kubectl get networkpolicy -A
```

### 6.3. 계층별 진단 접근법

네트워크 문제가 발생하면 상위 레이어부터 하위 레이어로 순서대로 진단합니다.

```
L7: Application  (Pod 응답 확인)
L4: Service/Endpoint (kube-proxy, iptables)
L3: IP 라우팅  (Calico, BGP 경로)
L2: Node 네트워크  (노드 간 ping)
L1: 물리 네트워크

→ Pod/Service가 정상인데 통신 실패 시 L3(Calico BGP) 확인!
```

### 6.4. 트러블슈팅 체크리스트

```
□ Pod 상태 확인 (Running, Ready)
□ Service/Endpoint 확인
□ Node 상태 확인 (Ready)
□ NetworkPolicy 확인 (kubectl get networkpolicy -A)
□ 노드 간 ping 테스트
□ 라우팅 테이블 확인 (ip route)
□ Calico Pod 상태 확인
□ Calico 로그 확인 (ERROR, BGP)
□ BGP Peering 상태 확인 (calicoctl node status)
□ 방화벽/Security Group 포트 179 확인
```

---

## 7. 핵심 명령어 요약

```bash
# BGP Peering 상태 확인
kubectl exec -n kube-system <calico-pod> -c calico-node -- calicoctl node status

# BGP Peer 상세 정보
kubectl exec -n kube-system <calico-pod> -c calico-node -- calicoctl get bgppeer -o yaml

# 라우팅 테이블 확인
kubectl exec -n kube-system <calico-pod> -c calico-node -- ip route

# IP Pool 확인
kubectl exec -n kube-system <calico-pod> -- calicoctl get ippool -o wide

# Node 정보 확인
kubectl exec -n kube-system <calico-pod> -- calicoctl get node -o wide

# calico-node 개별 재시작
kubectl delete pod -n kube-system <calico-pod-name>

# calico-node 전체 순차 재시작 (30초 간격)
kubectl get pods -n kube-system -l k8s-app=calico-node -o name | \
  xargs -I {} sh -c 'kubectl delete -n kube-system {} && sleep 30'

# calico-node 재시작 모니터링
kubectl get pods -n kube-system -l k8s-app=calico-node -w
```

---

## 8. 운영 참고사항

**BGP 포트**: TCP **179** (방화벽/Security Group에서 반드시 허용)

**Peering 확인 주기**: 기본 60초 (keepalive)

**Peering 방식 선택 기준**:
- 50개 이하: Node-to-Node Mesh (기본값)
- 50~100개: Node-to-Node Mesh 또는 Route Reflector 검토
- 100개 이상: Route Reflector 전환 권장

**Prometheus Alert 설정**:

```yaml
- alert: CalicoBGPPeeringDown
  expr: calico_bgp_peers_state{state="down"} > 0
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Calico BGP Peering is down"
    description: "BGP peer {{ $labels.peer }} is down on node {{ $labels.node }}"
```

**금융권 환경 주의사항**:
- BGP Peering 로그 모니터링 필수 (네트워크 안정성)
- Calico 버전을 최신 안정 버전으로 유지 (구버전은 BGP Peering 불안정 이슈 존재)
- calico-node 순차 재시작 원칙 준수 (동시 재시작 금지)

---

## 참고 자료

- Calico 공식 문서: <https://docs.tigera.io/calico/latest/networking/bgp>
- 실제 장애 사례: [[2026-02-06] MinIO Calico BGP Peering 장애](/posts/MinIO-Calico-BGP-Peering)