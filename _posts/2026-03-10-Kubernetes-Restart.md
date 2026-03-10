---
title: "[Kubernetes] Rollout & Pod/Container 재시작"
date: 2026-03-10 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, rollout, deployment, daemonset, pod]
render_with_liquid: false
---

## 📌 개요

**한 줄 요약**: Deployment, DaemonSet 등의 Pod를 안전하게 교체하는 Rolling Update 메커니즘

**핵심 개념**:
- **Pod 재시작**: Pod 삭제 후 재생성 (IP 변경, AGE 초기화)
- **컨테이너 재시작**: Pod는 유지, 컨테이너만 재시작 (RESTARTS 증가)
- **Rollout**: spec 변경 시 순차적으로 Pod를 안전하게 교체

---

## 🔄 Pod 재시작 vs 컨테이너 재시작

### Pod 재시작 (Pod 삭제 후 재생성)
```
kubectl delete pod <pod-name>
    ↓
Pod 완전히 삭제
    ↓
새 Pod 생성 (새로운 IP, 새로운 AGE)
    ↓
kubectl get pods → AGE가 짧아짐
```

**특징**:
- Pod IP 변경
- AGE 초기화
- RESTARTS 0으로 초기화
- 새로운 리소스 할당

**발생 원인**:
- `kubectl delete pod`
- 노드 장애
- Deployment 이미지 변경
- `kubectl rollout restart`

### 컨테이너 재시작 (Pod는 유지)
```
컨테이너 프로세스 비정상 종료 (OOMKill, Crash 등)
    ↓
Pod는 그대로 유지
    ↓
컨테이너만 재시작
    ↓
kubectl get pods → AGE 그대로, RESTARTS 증가
```

**특징**:
- Pod IP 유지
- AGE 유지
- RESTARTS 숫자 누적 증가
- 동일한 리소스 재사용

**발생 원인**:
- OOMKilled (메모리 초과)
- CrashLoopBackOff (프로세스 비정상 종료)
- Liveness Probe 실패
- 컨테이너 에러

### 비교표

| 항목 | Pod 재시작 | 컨테이너 재시작 |
|------|-----------|----------------|
| **AGE** | 변경됨 (초기화) | 그대로 유지 |
| **IP** | 변경됨 | 그대로 유지 |
| **RESTARTS** | 0으로 초기화 | 누적 증가 |
| **원인** | kubectl delete, 노드 장애, rollout | OOMKill, Crash, Probe 실패 |
| **확인** | `kubectl get pods` → AGE 확인 | `kubectl get pods` → RESTARTS 확인 |

---

## 🎯 Rollout 메커니즘

### 기본 원리
```
spec 변경 감지 (이미지 변경, annotation 변경 등)
    ↓
updateStrategy에 따라 순차적으로 Pod 교체
    ↓
새 Pod Running 확인 후 다음 Pod 교체
    ↓
모든 Pod 교체 완료
```

### rollout restart 동작
```bash
kubectl rollout restart daemonset -n kube-system calico-node
```

**내부 동작**:
```
1. DaemonSet annotation에 재시작 시간 추가
   kubectl.kubernetes.io/restartedAt: "2025-02-19T21:00:00"
    ↓
2. spec 변경으로 인식 → Rolling Update 트리거
    ↓
3. updateStrategy에 따라 순차 재시작
    ↓
4. 각 Pod:
   - 기존 Pod 삭제
   - 새 Pod 생성
   - Running 확인
   - 다음 Pod로 진행
```

---

## 🔧 updateStrategy

### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate  # 기본값
    rollingUpdate:
      maxUnavailable: 1  # 동시에 내릴 수 있는 Pod 수
      maxSurge: 1        # 추가로 올릴 수 있는 Pod 수
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

**동작 예시** (replicas: 5 → 이미지 변경):
```
1. 새 Pod 1개 생성 (maxSurge=1) → 총 6개
2. 새 Pod Running 확인
3. 기존 Pod 1개 삭제 (maxUnavailable=1) → 총 5개
4. 반복 → 모든 Pod 교체 완료
```

### DaemonSet (calico-node)
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: calico-node
  namespace: kube-system
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # 동시에 재시작할 노드 수
      maxSurge: 0        # DaemonSet은 노드당 1개라 Surge 없음
  template:
    spec:
      containers:
      - name: calico-node
        image: calico/node:v3.20
```

**DaemonSet 특징**:
- 노드당 Pod 1개만 실행
- `maxSurge: 0` (추가 Pod 불가)
- `maxUnavailable: 1` → 한 번에 1개 노드씩 재시작

### updateStrategy 타입

| type | 동작 | 사용 시기 |
|------|------|-----------|
| **RollingUpdate** | 순차적으로 하나씩 교체 (기본값) | 일반적인 업데이트 |
| **OnDelete** | 직접 삭제할 때만 교체 (자동 X) | 수동 제어 필요 시 |

---

## 💻 Rollout 명령어

### 기본 명령어
```bash
# 재시작 (annotation 변경 방식)
kubectl rollout restart deployment web-app -n default
kubectl rollout restart daemonset calico-node -n kube-system

# 진행 상황 확인
kubectl rollout status deployment web-app -n default
# Waiting for deployment "web-app" rollout to finish: 2 out of 5 new replicas have been updated...
# deployment "web-app" successfully rolled out

# 일시 중지 (문제 발생 시)
kubectl rollout pause deployment web-app -n default

# 재개
kubectl rollout resume deployment web-app -n default

# 히스토리 확인
kubectl rollout history deployment web-app -n default
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         kubectl set image deployment/web-app nginx=nginx:1.22

# 이전 버전으로 롤백
kubectl rollout undo deployment web-app -n default

# 특정 revision으로 롤백
kubectl rollout undo deployment web-app -n default --to-revision=1
```

### 실시간 모니터링
```bash
# Rollout 진행 상황 실시간 확인
watch kubectl get pods -n default

# DaemonSet 노드별 상태
kubectl get daemonset -n kube-system calico-node -o wide

# 이벤트 확인
kubectl get events -n default --sort-by='.lastTimestamp' | grep web-app
```

---

## 🔍 컨테이너 재시작 원인 분석

### 재시작 원인 확인
```bash
# 재시작 원인 확인
kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Last State"

# 출력 예시:
# Last State:     Terminated
#   Reason:       OOMKilled
#   Exit Code:    137
#   Started:      Wed, 19 Feb 2025 10:00:00 +0900
#   Finished:     Wed, 19 Feb 2025 10:05:00 +0900

# 이전 컨테이너 로그 확인
kubectl logs <pod-name> -n <namespace> --previous

# RESTARTS 높은 Pod 찾기
kubectl get pods -A --sort-by='.status.containerStatuses[0].restartCount' | tail -10
```

### 주요 재시작 원인

| 원인 | 증상 | 확인 방법 | 조치 |
|------|------|-----------|------|
| **OOMKilled** | 메모리 초과 | `Reason: OOMKilled` / `Exit Code: 137` | Memory Limit 증가 |
| **CrashLoopBackOff** | 프로세스 비정상 종료 | `Reason: Error` / `Exit Code: 1` | 로그 확인, 코드 수정 |
| **Liveness Probe 실패** | 헬스체크 실패 | `Liveness probe failed` | Probe 설정 확인 |
| **노드 리소스 부족** | 여러 Pod 동시 재시작 | `kubectl top nodes` | 노드 리소스 확인 |

### 디버깅 예시
```bash
# OOMKilled 확인
kubectl describe pod my-app-123 -n default | grep -i oom

# 리소스 사용량 확인
kubectl top pod my-app-123 -n default

# 이벤트 확인
kubectl get events -n default --field-selector involvedObject.name=my-app-123
```

---

## ⚠️ 주의사항

### Rollout 관련

**1. maxUnavailable 너무 크게 설정**
```yaml
# ❌ 위험
rollingUpdate:
  maxUnavailable: 5  # replicas 5개 중 5개 내림 → 서비스 중단

# ✅ 안전
rollingUpdate:
  maxUnavailable: 1  # 최소 가용성 보장
```

**2. Probe 미설정 시 문제**
```yaml
# ❌ Probe 없음
spec:
  containers:
  - name: app
    image: myapp:1.0
    # readinessProbe 없음 → 새 Pod가 준비 안 됐는데 트래픽 받음

# ✅ Probe 설정
spec:
  containers:
  - name: app
    image: myapp:1.0
    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
```

**3. DaemonSet maxUnavailable**
```yaml
# ❌ 위험 (노드 3개 클러스터)
maxUnavailable: 3  # 모든 노드 동시 재시작 → calico-node 전체 중단

# ✅ 안전
maxUnavailable: 1  # 한 번에 1개 노드씩
```

### 컨테이너 재시작 관련

**1. RESTARTS 계속 증가**
```bash
# 원인 파악
kubectl describe pod <pod-name> | grep -A 10 "Last State"

# CrashLoopBackOff → 코드 문제
# OOMKilled → 메모리 부족
# Liveness probe failed → Probe 설정 문제
```

**2. 메모리 부족 해결**
```yaml
# 현재 사용량 확인
kubectl top pod <pod-name>

# Memory Limit 증가
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "1Gi"  # 512Mi → 1Gi로 증가
```

---

## 📝 요약

### Pod vs 컨테이너 재시작
```
Pod 재시작:
- Pod 삭제 후 재생성
- IP 변경, AGE 초기화, RESTARTS 0
- 원인: kubectl delete, rollout, 노드 장애

컨테이너 재시작:
- Pod 유지, 컨테이너만 재시작
- IP 유지, AGE 유지, RESTARTS 증가
- 원인: OOMKill, Crash, Probe 실패
```

### Rollout 핵심
```
동작:
1. spec 변경 감지 (이미지, annotation)
2. updateStrategy에 따라 순차 교체
3. 새 Pod Running 확인 후 다음 진행

명령어:
kubectl rollout restart <type> <name>
kubectl rollout status <type> <name>
kubectl rollout pause / resume <type> <name>
kubectl rollout undo <type> <name>
kubectl rollout undo <type> <name> --to-revision=N

안전 설정:
- maxUnavailable: 1 (최소 가용성 유지)
- readinessProbe 필수
- DaemonSet은 maxUnavailable: 1 권장
```

---

## 🔗 관련 문서

- [공식 문서 - Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)