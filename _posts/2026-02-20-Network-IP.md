---
title: "[Network] 공인IP vs 사설IP, 그리고 Kubernetes 네트워크 구조 정리"
date: 2026-02-20 00:00:00 +0900
categories: [Network]
tags: [network, ip, nat, vpc, subnet, calico, bastion, hypercloud]
render_with_liquid: false
---

# 공인IP vs 사설IP

## 1. 개요

- **공인IP**: 인터넷에 직접 노출해야 하는 서비스 (웹서버, API 서버)에 사용
- **사설IP**: 내부 네트워크 통신, 보안이 필요한 서비스 (DB, 내부 API)에 사용
- **NAT/Proxy**: 사설IP를 공인IP로 변환하여 인터넷 접근

**관련 기술**: NAT (Network Address Translation), VPC, Subnet, 방화벽, Proxy

---

## 2. 핵심 개념

### 2.1. 공인IP (Public IP)

```
인터넷 → 공인IP → 서버
```

- 전 세계 어디서나 직접 접근 가능
- IANA에서 할당, ISP가 제공
- 유일성 보장 (중복 불가)

### 2.2. 사설IP (Private IP)

```
내부 네트워크 → 사설IP → 서버 → NAT → 공인IP → 인터넷
```

- 내부 네트워크에서만 사용
- 인터넷 직접 접근 불가
- 네트워크마다 동일 IP 재사용 가능

### 2.3. NAT (Network Address Translation)

```
사설IP (10.0.1.5) → NAT Gateway → 공인IP (1.2.3.4) → 인터넷
```

- 사설IP를 공인IP로 변환
- 여러 사설IP가 하나의 공인IP 공유
- 보안 및 IP 절약

### 2.4. 주요 차이점

| 항목 | 공인IP | 사설IP |
|------|--------|--------|
| **접근 범위** | 인터넷 전체 | 내부 네트워크만 |
| **유일성** | 전 세계 유일 | 네트워크마다 재사용 가능 |
| **할당 주체** | IANA → ISP | 네트워크 관리자 |
| **비용** | 유료 (ISP 요금) | 무료 |
| **보안** | 직접 노출 (위험) | 내부 격리 (안전) |
| **예시** | 1.2.3.4, 8.8.8.8 | 192.168.1.10, 10.0.1.5 |
| **사용 시나리오** | 웹서버, 공개 API | DB, 내부 서비스, K8s Pod |

### 2.5. 사설IP 대역 (RFC 1918)

| 클래스 | IP 범위 | CIDR | 사용 가능 IP 수 | 주 용도 |
|--------|---------|------|----------------|---------|
| **A** | 10.0.0.0 ~ 10.255.255.255 | 10.0.0.0/8 | 16,777,216개 | 대규모 기업, 클라우드 VPC |
| **B** | 172.16.0.0 ~ 172.31.255.255 | 172.16.0.0/12 | 1,048,576개 | 중규모 네트워크 |
| **C** | 192.168.0.0 ~ 192.168.255.255 | 192.168.0.0/16 | 65,536개 | 가정, 소규모 사무실 |

**특수 대역**:

- **127.0.0.0/8**: Loopback (localhost)
- **169.254.0.0/16**: Link-local (DHCP 실패 시 자동 할당)

---

## 3. 실무 사용법

### 3.1. 기본 명령어

```bash
# 사설IP 확인 (Linux)
hostname -I
ip addr show

# 공인IP 확인
curl ifconfig.me
curl ipinfo.io/ip

# 특정 인터페이스 IP 확인
ip addr show eth0

# 라우팅 테이블 확인
ip route

# DNS 조회
nslookup google.com
dig google.com

# 특정 IP 통신 가능 여부 확인
ping 8.8.8.8
ping 192.168.1.1

# 포트 연결 테스트
telnet 192.168.1.10 22
nc -zv 10.0.1.5 3306
```

**출력 예시**:

```
# hostname -I
192.168.1.10 10.0.1.5

# curl ifconfig.me
203.0.113.45

# ip addr show eth0
inet 10.0.1.5/24 brd 10.0.1.255 scope global eth0
```

---

## 4. 패턴별 구성 예시

### 패턴 1: AWS VPC 구성 (공인/사설 서브넷 분리)

```yaml
# VPC 구조
VPC: 10.0.0.0/16

  Public Subnet (10.0.1.0/24)
    - Bastion Host: 10.0.1.10 (공인IP: 203.0.113.45)
    - NAT Gateway: 10.0.1.20 (공인IP: 203.0.113.46)
    - LoadBalancer: 10.0.1.30 (공인IP: 203.0.113.47)
    → 인터넷 게이트웨이 연결

  Private Subnet (10.0.2.0/24)
    - K8s Master: 10.0.2.10
    - K8s Worker: 10.0.2.11, 10.0.2.12
    - Database: 10.0.2.20
    → NAT Gateway를 통해 인터넷 접근
```

**라우팅 테이블**:

```
# Public Subnet
목적지: 0.0.0.0/0 → 인터넷 게이트웨이
목적지: 10.0.0.0/16 → Local

# Private Subnet
목적지: 0.0.0.0/0 → NAT Gateway (10.0.1.20)
목적지: 10.0.0.0/16 → Local
```

**접근 시나리오**:

1. 외부 사용자 → 공인IP (203.0.113.47) → LoadBalancer (10.0.1.30)
2. LoadBalancer → Private Subnet Pod (10.0.2.50)
3. Pod → NAT Gateway (10.0.1.20) → 인터넷 (외부 API 호출)

---

### 패턴 2: Kubernetes 클러스터 네트워크 (HyperCloud 환경)

```yaml
AWS VPC: 10.100.0.0/16

  K8s Master Nodes (Public Subnet)
    - Master-1: 10.100.124.80 (공인IP: ELB)
    - Master-2: 10.100.124.81
    - Master-3: 10.100.124.82

  K8s Worker Nodes (Private Subnet)
    - Worker-1: 10.100.142.110
    - Worker-2: 10.100.142.111
    - Worker-3: 10.100.130.60

  Pod Network (Calico CNI)
    - Pod CIDR: 10.244.0.0/16
    - Pod-1: 10.244.1.5
    - Pod-2: 10.244.2.10
    → 사설IP, Service를 통해 접근

  Service Network
    - Service CIDR: 10.96.0.0/12
    - ClusterIP: 10.96.100.50
    - NodePort: 워커노드IP:31234
```

**접근 경로**:

```
외부 사용자
  ↓
공인 도메인 (app.example.com)
  ↓
AWS ELB (공인IP)
  ↓
Traefik Ingress (10.244.1.5)
  ↓
Service (ClusterIP: 10.96.100.50)
  ↓
Pod (10.244.2.10)
```

---

### 패턴 3: 금융권 온프레미스 + 클라우드 하이브리드

```yaml
# 온프레미스 (내부망)
사내 네트워크: 172.16.0.0/16
  - DB 서버: 172.16.1.10
  - 레거시 시스템: 172.16.2.20
  → 인터넷 접근 불가 (폐쇄망)

# AWS VPC (클라우드)
VPC: 10.0.0.0/16
  - K8s 클러스터: 10.0.2.0/24
  - API Gateway: 10.0.1.30 (공인IP: 203.0.113.50)

# VPN/Direct Connect
온프레미스 (172.16.0.0/16) ←→ AWS VPC (10.0.0.0/16)
```

**라우팅**:

```
# AWS VPC 라우팅 테이블
목적지: 172.16.0.0/16 → VPN Gateway
목적지: 0.0.0.0/0    → 인터넷 게이트웨이

# 온프레미스 라우팅
목적지: 10.0.0.0/16  → VPN Tunnel
목적지: 0.0.0.0/0    → 차단 (폐쇄망)
```

---

### 패턴 4: NAT Gateway를 통한 인터넷 접근

```bash
# Private Subnet의 Pod가 외부 API 호출 시 동작 흐름
# Pod (10.244.1.10) → 외부 API 요청
kubectl exec -it api-client -- curl https://api.github.com
# 출발: 10.244.1.10
# NAT Gateway에서 공인IP로 변환: 203.0.113.46
# GitHub이 보는 IP: 203.0.113.46
```

**동작 흐름**:

1. Pod (10.244.1.10) → 외부 API 요청
2. Calico CNI → Worker Node (10.0.2.11)로 라우팅
3. Worker Node → NAT Gateway (10.0.1.20)로 전달
4. NAT Gateway → 공인IP (203.0.113.46)로 변환
5. 인터넷 → 외부 API 서버

---

### 패턴 5: Bastion Host (점프 서버)

```bash
# 1. Bastion Host (공인IP: 203.0.113.45, 사설IP: 10.0.1.10) 접속
ssh -i bastion-key.pem ec2-user@203.0.113.45

# 2. Bastion에서 Private 인스턴스 접속 (사설IP만 존재)
ssh -i worker-key.pem ec2-user@10.0.2.11

# SSH 터널링 (로컬 → Bastion → Private)
ssh -i bastion-key.pem -L 6443:10.0.2.10:6443 ec2-user@203.0.113.45
# 이제 로컬의 localhost:6443 → K8s API Server (10.0.2.10:6443) 접근 가능
```

---

### 패턴 6: Service 타입별 IP 사용

```yaml
# ClusterIP (사설IP만, 내부 통신)
apiVersion: v1
kind: Service
metadata:
  name: internal-api
spec:
  type: ClusterIP
  clusterIP: 10.96.100.50  # 사설IP (Service CIDR)
  selector:
    app: api
  ports:
  - port: 8080
# → 클러스터 내부에서만 접근 가능

---
# NodePort (사설IP + 노드포트)
apiVersion: v1
kind: Service
metadata:
  name: external-api
spec:
  type: NodePort
  selector:
    app: api
  ports:
  - port: 8080
    nodePort: 31234
# → 워커노드IP:31234로 외부 접근 가능
# → 노드IP가 사설IP면 VPN 필요

---
# LoadBalancer (공인IP 자동 할당)
apiVersion: v1
kind: Service
metadata:
  name: public-web
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
# → AWS ELB 자동 생성, 공인IP 할당
# → 인터넷에서 직접 접근 가능
```

---

## 5. 주의사항 및 흔한 실수

### 공인IP를 사설IP 대역에 할당

```
❌ EC2 인스턴스에 192.168.1.10 공인IP 할당 시도
→ 불가능, 192.168.x.x는 사설IP 대역

✅ 공인IP는 ISP에서 할당받은 IP만 사용
```

### 사설IP로 직접 인터넷 접근 시도

```
❌ curl http://10.0.1.5 (외부에서)
→ 연결 불가, 사설IP는 라우팅 안 됨

✅ NAT Gateway 또는 VPN 필요
```

### NAT Gateway 없이 Private Subnet에서 외부 접근

```
❌ Private Pod에서 apt update 실패
→ 인터넷 접근 불가

✅ NAT Gateway 생성 및 라우팅 테이블 설정
```

### Security Group에서 사설IP 차단

```
❌ Inbound 규칙: 0.0.0.0/0 차단
→ VPC 내부 통신도 차단됨

✅ VPC CIDR (10.0.0.0/16) 허용
```

### Bastion 없이 Private 인스턴스 접근

```
❌ ssh ec2-user@10.0.2.11 (로컬에서)
→ 연결 불가, Private IP는 VPC 외부에서 라우팅 안 됨

✅ Bastion Host 또는 VPN 사용
```

### 공인IP 하드코딩

```
❌ 애플리케이션 코드에 공인IP 직접 입력
→ 공인IP 변경 시 코드 수정 필요

✅ DNS 사용 (app.example.com)
```

### VPC 피어링 시 IP 대역 중복

```
❌ VPC-A: 10.0.0.0/16, VPC-B: 10.0.0.0/16
→ 피어링 시 라우팅 충돌

✅ 서로 다른 대역 사용
   VPC-A: 10.0.0.0/16, VPC-B: 10.1.0.0/16
```

---

## 6. 트러블슈팅 워크플로우

```bash
# 시나리오: Private Subnet Pod에서 외부 API 호출 실패

# Step 1: Pod IP 확인
kubectl exec -it api-client -- hostname -I
# 10.244.1.10 (사설IP)

# Step 2: 인터넷 연결 테스트
kubectl exec -it api-client -- ping -c 3 8.8.8.8
# 실패 → NAT Gateway 문제

# Step 3: Worker Node에서 테스트
ssh worker-node-1
ping 8.8.8.8
# 성공 → 노드는 인터넷 접근 가능

# Step 4: Calico 정책 확인
kubectl get networkpolicy -n <namespace>
# Egress 차단 규칙 있는지 확인

# Step 5: NAT Gateway 확인 (AWS Console)
# VPC Console → NAT Gateways → State: Available, Elastic IP 연결 확인

# Step 6: 라우팅 테이블 확인
# Route Tables → Private Subnet → 0.0.0.0/0 → NAT Gateway 확인

# Step 7: Security Group 확인
# Worker Node Security Group → Outbound: 0.0.0.0/0 All traffic 허용 확인

# Step 8: Pod IP 및 라우팅 확인
kubectl exec -it <pod-name> -- ip addr
kubectl exec -it <pod-name> -- ip route

# Step 9: Service Endpoint 확인
kubectl get endpoints <service-name>
```

---

## 7. 금융권 환경 고려사항

**내부망 (폐쇄망)**:

```
사설IP만 사용: 172.16.0.0/16
인터넷 접근: 차단
외부 통신: VPN 또는 전용선만
```

**DMZ (외부망)**:

```
공인IP 사용: ISP 할당
인터넷 접근: 허용 (방화벽 통과)
내부망 접근: 방화벽 규칙으로 제한
```

**클라우드 (하이브리드)**:

```
VPC: 10.0.0.0/16
Public Subnet: LoadBalancer, Bastion
Private Subnet: K8s, DB
온프레미스 연결: Direct Connect
```

---

## 8. 참고

- RFC 1918 (Private IP): <https://datatracker.ietf.org/doc/html/rfc1918>
- AWS VPC 공식 문서: <https://docs.aws.amazon.com/vpc/>

**HyperCloud 환경 요약**:

- Master Node: Public Subnet (ELB 통해 접근)
- Worker Node: Private Subnet (NAT Gateway 통해 인터넷 접근)
- Pod: Calico CNI (10.244.0.0/16)

**Kubernetes Service Type 선택 기준**:

- 내부 통신만: ClusterIP (사설IP)
- 개발/테스트: NodePort (노드 사설IP + 포트)
- 프로덕션: LoadBalancer (공인IP 자동 할당) 또는 Ingress