---
title: "[Kubernetes] ReplicaSet"
date: 2026-03-10 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, replicaset, deployment, pod]
render_with_liquid: false
---

## 📌 개요

**한 줄 요약**: 지정한 수의 Pod가 항상 실행되도록 보장하는 리소스

**핵심 동작**:
```
replicas: 3 설정
    ↓
ReplicaSet이 항상 3개 Pod 유지
    ↓
Pod 1개 죽으면 → 자동으로 새 Pod 생성
Pod 1개 수동 추가되면 → 자동으로 1개 삭제
```

**실무 사용**:
- ❌ ReplicaSet 직접 생성 (비권장)
- ✅ Deployment를 통해 간접 관리 (권장)

---

## 🏗 Deployment와의 관계

### 계층 구조
```
Deployment (배포 관리)
    ↓ 생성/관리
ReplicaSet (Pod 수 관리)
    ↓ 생성/관리
Pod (실제 컨테이너)
```

### Deployment vs ReplicaSet

| 기능 | Deployment | ReplicaSet |
|------|-----------|-----------|
| **롤링 업데이트** | ✅ 지원 | ❌ 불가능 |
| **롤백** | ✅ 지원 | ❌ 불가능 |
| **Pod 수 보장** | ✅ (RS 통해) | ✅ 지원 |
| **이력 관리** | ✅ 지원 | ❌ 불가능 |
| **직접 사용** | ✅ 권장 | ❌ 비권장 |

**실무 권장**:
```yaml
# ❌ ReplicaSet 직접 생성 (하지 마세요)
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-app-rs
spec:
  replicas: 3
  # ...

# ✅ Deployment 사용 (권장)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  # Deployment가 ReplicaSet 자동 생성
```

---

## 🔄 ReplicaSet 생성 시점

### 새 ReplicaSet이 생성되는 경우

Deployment의 `template.spec` 내용이 바뀌면 새 ReplicaSet이 생성됩니다.

| 변경 사항 | 새 RS 생성 | 이유 |
|----------|----------|------|
| **이미지 변경** | ✅ | Pod spec 변경 |
| **환경변수 변경** | ✅ | Pod spec 변경 |
| **Requests/Limits 변경** | ✅ | Pod spec 변경 |
| **Volume 변경** | ✅ | Pod spec 변경 |
| **Command/Args 변경** | ✅ | Pod spec 변경 |
| **replicas 변경** | ❌ | 기존 RS 수정 |
| **metadata.labels 변경** | ❌ | Pod spec 무관 |

### 예시: 이미지 변경
```yaml
# 초기 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21  # 버전 1
```

**이미지 변경**:
```bash
kubectl set image deployment/web-app nginx=nginx:1.22
```

**결과**:
```
기존: web-app-7d4b5c8f9d (nginx:1.21) → 0/3
새로: web-app-5f6g7h8i9j (nginx:1.22) → 3/3
```

### 예시: replicas 변경
```bash
kubectl scale deployment web-app --replicas=5
```

**결과**:
```
기존: web-app-7d4b5c8f9d → 5/5 (새 RS 생성 안 함)
```

---

## 🚀 롤링 업데이트 동작

### 업데이트 흐름
```
1. Deployment template.spec 변경
    ↓
2. 새 ReplicaSet 생성 (0/3)
    ↓
3. 새 ReplicaSet Pod 하나씩 늘어남 (1/3, 2/3, 3/3)
    ↓
4. 기존 ReplicaSet Pod 하나씩 줄어듦 (2/3, 1/3, 0/3)
    ↓
5. 기존 ReplicaSet은 0/3으로 보존 (삭제 안 함!)
```

### 단계별 상태
```
초기 상태:
ReplicaSet-v1 (nginx:1.21): 3/3
ReplicaSet-v2 (nginx:1.22): 없음

업데이트 시작:
ReplicaSet-v1: 3/3
ReplicaSet-v2: 0/3 (생성)

진행 중 (1):
ReplicaSet-v1: 2/3
ReplicaSet-v2: 1/3

진행 중 (2):
ReplicaSet-v1: 1/3
ReplicaSet-v2: 2/3

업데이트 완료:
ReplicaSet-v1: 0/3 (보존 ✅)
ReplicaSet-v2: 3/3
```

### 실시간 확인
```bash
# 업데이트 진행 상황 모니터링
kubectl rollout status deployment web-app -n default

# ReplicaSet 변화 실시간 확인
watch kubectl get rs -n default

# 출력 예시:
NAME                DESIRED   CURRENT   READY   AGE
web-app-7d4b5c8f9d  0         0         0       10m  # 구버전
web-app-5f6g7h8i9j  3         3         3       2m   # 신버전
```

---

## 🔙 기존 ReplicaSet 보존 이유

### 롤백을 위한 보존

기존 ReplicaSet을 0/3 상태로 보존하는 이유는 **롤백 기능** 때문입니다.
```
배포 이력:
Revision 1: ReplicaSet-v1 (nginx:1.21) → 0/3 보존
Revision 2: ReplicaSet-v2 (nginx:1.22) → 0/3 보존
Revision 3: ReplicaSet-v3 (nginx:1.23) → 3/3 현재
    ↓
롤백 가능!
```

### 롤백 명령어
```bash
# 히스토리 확인
kubectl rollout history deployment web-app -n default

# 출력:
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/web-app nginx=nginx:1.22
3         kubectl set image deployment/web-app nginx=nginx:1.23

# 특정 버전 상세 확인
kubectl rollout history deployment web-app -n default --revision=2

# 바로 이전 버전으로 롤백 (Revision 2)
kubectl rollout undo deployment web-app -n default

# 특정 버전으로 롤백 (Revision 1)
kubectl rollout undo deployment web-app -n default --to-revision=1
```

### 롤백 동작
```
롤백 실행 (Revision 3 → Revision 2)
    ↓
기존 ReplicaSet-v2 (0/3) Pod 다시 늘어남
ReplicaSet-v2: 0/3 → 1/3 → 2/3 → 3/3
    ↓
현재 ReplicaSet-v3 (3/3) Pod 줄어듦
ReplicaSet-v3: 3/3 → 2/3 → 1/3 → 0/3 (보존)
    ↓
롤백 완료
ReplicaSet-v2: 3/3 (활성)
ReplicaSet-v3: 0/3 (보존)
```

---

## ⚙️ 보존 개수 제한

### revisionHistoryLimit

기본적으로 최근 **10개**의 ReplicaSet을 보존합니다.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  revisionHistoryLimit: 10  # 기본값
  # ...
```

**동작**:
```
Revision 1~10: 보존
Revision 11 생성 → Revision 1 자동 삭제 (오래된 것부터)
Revision 12 생성 → Revision 2 자동 삭제
```

**값 조정**:
```yaml
# 적게 보존 (디스크 절약)
revisionHistoryLimit: 3

# 많이 보존 (롤백 옵션 확대)
revisionHistoryLimit: 20

# 보존 안 함 (롤백 불가, 비권장)
revisionHistoryLimit: 0
```

---

## 🔍 ReplicaSet 확인

### 기본 명령어
```bash
# ReplicaSet 목록
kubectl get rs -n default

# 출력:
NAME                DESIRED   CURRENT   READY   AGE
web-app-7d4b5c8f9d  0         0         0       1h   # 구버전 (보존)
web-app-5f6g7h8i9j  3         3         3       10m  # 신버전 (활성)

# ReplicaSet 상세 정보
kubectl describe rs web-app-5f6g7h8i9j -n default

# Deployment와 연결된 ReplicaSet 확인
kubectl get rs -n default -l app=web-app

# ReplicaSet이 관리하는 Pod 확인
kubectl get pods -n default -l app=web-app
```

### ReplicaSet 정보 분석
```bash
# ReplicaSet의 Pod Template Hash 확인
kubectl get rs web-app-5f6g7h8i9j -n default -o yaml | grep pod-template-hash

# Deployment에서 현재 활성 ReplicaSet 확인
kubectl get deployment web-app -n default -o jsonpath='{.spec.selector}'

# 특정 ReplicaSet의 소유자 확인 (Deployment)
kubectl get rs web-app-5f6g7h8i9j -n default -o jsonpath='{.metadata.ownerReferences[0].name}'
```

---

## ⚠️ 주의사항

### ReplicaSet 직접 수정 금지
```bash
# ❌ ReplicaSet 직접 수정하지 마세요
kubectl edit rs web-app-5f6g7h8i9j

# 이유:
# - Deployment가 다시 덮어씀
# - 변경사항 유지 안 됨

# ✅ Deployment를 수정하세요
kubectl edit deployment web-app
```

### ReplicaSet 직접 삭제 주의
```bash
# ❌ 활성 ReplicaSet 삭제 (Pod 전부 삭제됨!)
kubectl delete rs web-app-5f6g7h8i9j

# Deployment가 즉시 새 ReplicaSet 생성
# → 모든 Pod 재생성 → 서비스 중단!

# ✅ 구버전(0/3) ReplicaSet 삭제는 가능
kubectl delete rs web-app-7d4b5c8f9d
# → 롤백 불가능해짐
```

### 수동으로 생성한 Pod 문제
```bash
# ReplicaSet과 동일한 Label을 가진 Pod 수동 생성
kubectl run manual-pod --image=nginx --labels=app=web-app

# ReplicaSet이 감지 → 즉시 삭제!
# 이유: replicas=3인데 Pod가 4개 → 1개 삭제
```

---

## 📝 요약

### 핵심 개념
```
ReplicaSet:
- 지정한 수의 Pod 항상 유지
- Pod 장애 시 자동 재생성
- Deployment가 자동 생성/관리

Deployment → ReplicaSet → Pod:
- Deployment: 배포 관리 (롤링 업데이트, 롤백)
- ReplicaSet: Pod 수 보장
- Pod: 실제 컨테이너
```

### 새 ReplicaSet 생성 조건
```
✅ 생성:
- template.spec 변경 (이미지, 환경변수, 리소스, Volume 등)
- rollout restart 실행 시

❌ 미생성:
- replicas 변경
- metadata.labels 변경
- rollout undo (기존 RS의 replicas를 0 → n으로 올리는 것)
```

### 롤백 메커니즘
```
기존 ReplicaSet 0/3 보존:
- 롤백을 위한 이력 관리
- revisionHistoryLimit: 10 (기본값)
- 오래된 것부터 자동 삭제

롤백 명령어:
kubectl rollout undo deployment <name>
kubectl rollout undo deployment <name> --to-revision=2
```

### 권장 사항
```
✅ DO:
- Deployment 사용
- Deployment로 수정
- 구버전 RS는 자동 관리에 맡김

❌ DON'T:
- ReplicaSet 직접 생성
- ReplicaSet 직접 수정
- 활성 ReplicaSet 삭제
```

---

## 🔗 관련 문서

- [공식 문서 - ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)