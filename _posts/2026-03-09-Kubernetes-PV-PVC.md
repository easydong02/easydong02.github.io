---
title: "[Kubernetes] PV & PVC 완전 정리: 영구 스토리지 관리와 실무 패턴"
date: 2026-03-09 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, pv, pvc, storage, persistent-volume, storageclass, statefulset, nfs, efs, volume-snapshot]
render_with_liquid: false
---

# PV & PVC (Persistent Volume & Persistent Volume Claim)

## 1. 개요

**한 줄 요약**: Kubernetes에서 영구 스토리지를 관리하는 리소스 — PV는 실제 스토리지, PVC는 스토리지 요청서

**핵심 개념**:
- **PV (PersistentVolume)**: 관리자가 프로비저닝한 실제 스토리지 (물리 디스크, NFS, EBS 등)
- **PVC (PersistentVolumeClaim)**: 사용자가 요청하는 스토리지 (크기, 접근 모드)
- **바인딩**: PVC가 조건에 맞는 PV를 자동으로 찾아 1:1 연결

**비유**:

```
PV  = 아파트 (실제 부동산)
PVC = 임대 계약서 (아파트 요청 조건)
Pod = 입주자
```

**관련 기술**: StorageClass, StatefulSet, CSI Driver, VolumeSnapshot

---

## 2. 기본 구조

### 2.1. 관계도

```
┌─────────────┐
│    Pod      │
│  volumeMounts: /data
└──────┬──────┘
       │ 마운트
       ▼
┌─────────────┐
│    PVC      │  ← 사용자가 생성 (10Gi, ReadWriteOnce 요청)
│  my-pvc     │
└──────┬──────┘
       │ 바인딩 (자동)
       ▼
┌─────────────┐
│     PV      │  ← 관리자가 생성 또는 StorageClass가 자동 생성
│  my-pv      │
│  10Gi       │
│  EBS Volume │
└─────────────┘
```

### 2.2. 생명주기

```
1. Provisioning (프로비저닝)
   - Static:  관리자가 PV 미리 생성
   - Dynamic: StorageClass가 PVC 요청 시 PV 자동 생성

2. Binding (바인딩)
   - PVC 생성 → 조건 맞는 PV 찾기 → 1:1 바인딩

3. Using (사용)
   - Pod가 PVC를 volume으로 마운트

4. Reclaiming (회수)
   - PVC 삭제 시 PV 처리 방식 결정 (Retain / Delete / Recycle)
```

---

## 3. 주요 속성

### 3.1. Access Modes (접근 모드)

| 모드 | 약자 | 설명 | 사용 예시 |
|------|------|------|-----------|
| **ReadWriteOnce** | RWO | 단일 노드에서 읽기/쓰기 | EBS, Azure Disk |
| **ReadOnlyMany** | ROX | 여러 노드에서 읽기 전용 | 공유 설정 파일 |
| **ReadWriteMany** | RWX | 여러 노드에서 읽기/쓰기 | NFS, EFS, CephFS |

대부분의 블록 스토리지(EBS, Azure Disk)는 **RWO만 지원**합니다. 여러 Pod가 동시에 쓰기 접근이 필요하다면 NFS나 EFS를 사용해야 합니다.

### 3.2. Reclaim Policy (회수 정책)

| 정책 | 동작 | 사용 시기 |
|------|------|-----------|
| **Retain** | PVC 삭제해도 PV 유지, 데이터 보존 | 프로덕션 (수동 정리) |
| **Delete** | PVC 삭제 시 PV와 실제 스토리지 삭제 | 개발/테스트 (자동 정리) |
| **Recycle** | 데이터 삭제 후 PV 재사용 | Deprecated, 사용 안 함 |

### 3.3. Volume Binding Mode

| 모드 | 동작 | 사용 시기 |
|------|------|-----------|
| **Immediate** | PVC 생성 즉시 PV 바인딩 | EFS, NFS (노드 위치 무관) |
| **WaitForFirstConsumer** | Pod 생성 시점에 바인딩 | EBS (노드 위치 중요) |

---

## 4. 기본 사용법

### 4.1. Static Provisioning (수동)

```yaml
# Step 1. PV 생성 (관리자)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:              # 로컬 디스크 (테스트용)
    path: /mnt/data
---
# Step 2. PVC 생성 (사용자)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
---
# Step 3. Pod에서 사용
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: data
      mountPath: /usr/share/nginx/html
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
```

### 4.2. Dynamic Provisioning (자동) — 실무 권장

PVC만 생성하면 StorageClass가 PV를 자동으로 생성·바인딩합니다.

```yaml
# PVC만 생성 (PV는 자동 생성됨)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2    # AWS EBS gp2 StorageClass
  resources:
    requests:
      storage: 20Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: postgres:13
    volumeMounts:
    - name: db-data
      mountPath: /var/lib/postgresql/data
  volumes:
  - name: db-data
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

**동작 순서**:

```
1. PVC 생성 → StorageClass(gp2) 참조
2. StorageClass가 AWS EBS 20Gi 디스크 자동 생성
3. PV 자동 생성 및 PVC와 바인딩
4. Pod가 PVC를 통해 볼륨 마운트
```

---

## 5. 기본 명령어

```bash
# PV 전체 목록 확인
kubectl get pv

# PVC 확인
kubectl get pvc -n <namespace>

# PVC 상세 (바인딩 상태 및 이벤트 확인)
kubectl describe pvc <pvc-name> -n <namespace>

# PVC를 사용 중인 Pod 확인
kubectl get pods -n <namespace> -o json \
  | jq '.items[] | select(.spec.volumes[]?.persistentVolumeClaim.claimName=="my-pvc") | .metadata.name'

# PVC 삭제 (Pod 먼저 삭제 필요)
kubectl delete pod <pod-name> -n <namespace>
kubectl delete pvc <pvc-name> -n <namespace>

# PV 상세 확인
kubectl get pv <pv-name> -o yaml
```

**PV 상태 출력 예시**:

```
NAME    CAPACITY  ACCESS MODES  RECLAIM POLICY  STATUS    CLAIM            STORAGECLASS
my-pv   10Gi      RWO           Retain          Bound     default/my-pvc   manual

# STATUS 의미:
# Available : 사용 가능 (바인딩 안 됨)
# Bound     : PVC와 바인딩됨
# Released  : PVC 삭제됐지만 Retain 정책으로 PV 유지 중
# Failed    : 에러 상태
```

---

## 6. 실무 패턴

### 6.1. StatefulSet + volumeClaimTemplates

Pod마다 독립적인 PVC를 자동으로 생성합니다.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:       # Pod별 PVC 자동 생성
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: gp2
      resources:
        requests:
          storage: 50Gi
```

```
결과: PVC 자동 생성
  data-postgres-0  → Pod postgres-0 전용
  data-postgres-1  → Pod postgres-1 전용
  data-postgres-2  → Pod postgres-2 전용
```

### 6.2. 공유 스토리지 (ReadWriteMany)

여러 Pod가 동일한 파일시스템을 공유해야 할 때 EFS(AWS) 또는 NFS를 사용합니다.

```yaml
# EFS CSI Driver 사용 (AWS)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage
spec:
  accessModes:
    - ReadWriteMany    # 여러 Pod에서 동시 접근
  storageClassName: efs-sc
  resources:
    requests:
      storage: 100Gi
---
# Deployment: 5개 Pod가 동일 PVC 공유
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        persistentVolumeClaim:
          claimName: shared-storage    # 모든 Pod가 동일 PVC 사용
```

### 6.3. VolumeSnapshot (백업 & 복구)

```yaml
# PVC 백업
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: db-backup-20240104
spec:
  volumeSnapshotClassName: csi-aws-vsc
  source:
    persistentVolumeClaimName: db-pvc
---
# Snapshot에서 PVC 복구
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-restored
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp2
  dataSource:
    name: db-backup-20240104
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  resources:
    requests:
      storage: 50Gi
```

### 6.4. 동일 PVC를 여러 경로에 마운트 (1 볼륨 2 경로)

같은 PVC를 하나의 Pod 안에서 여러 `mountPath`로 마운트할 수 있습니다. **같은 방에 문을 2개 만드는 것**과 같습니다.

```yaml
volumes:
- name: my-vol
  persistentVolumeClaim:
    claimName: my-pvc           # PVC 1개

volumeMounts:
- name: my-vol
  mountPath: /LOGS              # 경로 1
- name: my-vol
  mountPath: /app/logs          # 경로 2 (같은 volume)
```

**동작 결과**:

```
실제 스토리지 (PVC):
/
├── file1.log
└── file2.log

컨테이너 내부:
/LOGS/           ← 동일한 데이터
├── file1.log
└── file2.log

/app/logs/       ← 동일한 데이터
├── file1.log
└── file2.log
```

두 경로는 **완전히 동일한 데이터**를 가리키며 실제 디스크 용량은 1배만 사용합니다. `/LOGS`에서 파일을 생성하면 `/app/logs`에서도 즉시 보이고, 한쪽에서 삭제하면 양쪽에서 모두 사라집니다.

**이렇게 쓰는 이유**:
- 레거시 호환: 옛날 경로(`/LOGS`)와 새 경로(`/app/logs`)를 동시에 지원
- 다른 프로세스: 같은 Pod 내 여러 프로세스가 서로 다른 경로를 기대할 때
- 심볼릭 링크 대체: `ln -s` 대신 volumeMount로 처리

```bash
# 동작 검증
kubectl exec <pod> -n <namespace> -- touch /LOGS/test.log
kubectl exec <pod> -n <namespace> -- ls /app/logs/
# test.log 보임 ✅
```

**주의**: 동시 쓰기 시 충돌 가능, 한쪽 삭제 시 양쪽 모두 삭제됨, 권한도 공유됨

---

## 7. 트러블슈팅

### 7.1. PVC가 Pending 상태

```bash
# 원인 확인
kubectl describe pvc <pvc-name> -n <namespace>

# 주요 원인 및 해결:
# 1. 조건 맞는 PV 없음    → PV 생성 또는 StorageClass 확인
# 2. StorageClass 없음   → kubectl get storageclass
# 3. 용량 부족           → PV 용량 증설
# 4. AccessMode 불일치   → PV/PVC Access Mode 맞추기

# PVC 이벤트 확인
kubectl get events -n <namespace> | grep -i pvc

# Available 상태 PV 확인
kubectl get pv | grep Available
```

### 7.2. PVC 삭제 안 됨

```bash
# Pod가 사용 중이면 삭제 불가 → Pod 먼저 삭제
kubectl delete pod <pod-name> -n <namespace>
kubectl delete pvc <pvc-name> -n <namespace>
```

### 7.3. PV Released 상태에서 재사용

```bash
# Retain 정책으로 PVC 삭제 후 Released 상태가 된 PV 재사용
kubectl edit pv <pv-name>
# spec.claimRef 항목 삭제 → Available 상태로 복귀
```

### 7.4. 흔한 실수

**데이터 손실 위험**

```yaml
# ❌ 프로덕션에서 Delete 정책 사용
persistentVolumeReclaimPolicy: Delete
# → PVC 삭제 시 실제 데이터도 삭제됨

# ✅ 프로덕션에서는 반드시 Retain 사용
persistentVolumeReclaimPolicy: Retain
```

**RWX를 RWO 스토리지에 요청**

```yaml
# ❌ EBS는 RWX 지원 안 함
accessModes:
  - ReadWriteMany

# ✅ EBS는 RWO만
accessModes:
  - ReadWriteOnce
# RWX가 필요하면 EFS, NFS, CephFS 사용
```

---

## 8. 운영 참고사항

### Static vs Dynamic Provisioning 비교

| 항목 | Static Provisioning | Dynamic Provisioning |
|------|---------------------|----------------------|
| **PV 생성** | 관리자가 수동 생성 | StorageClass가 자동 생성 |
| **사용 시기** | 레거시, 특수 스토리지 | 클라우드, 일반적 사용 |
| **장점** | 세밀한 제어 | 자동화, 편리 |
| **단점** | 수동 관리 부담 | StorageClass 의존 |

### 실무 권장 조합

- **프로덕션 DB**: Dynamic Provisioning + `reclaimPolicy: Retain`
- **StatefulSet**: `volumeClaimTemplates` 사용 (Pod별 독립 PVC)
- **공유 파일시스템**: ReadWriteMany + EFS / NFS StorageClass
- **백업**: VolumeSnapshot 주기적 실행

### 금융권 환경 고려사항

- **폐쇄망**: 클라우드 CSI Driver 대신 NFS 또는 Ceph 사용
- **데이터 보존**: 모든 중요 PV에 `reclaimPolicy: Retain` 적용
- **용량 모니터링**: `kubelet_volume_stats_used_bytes` 메트릭으로 PVC 사용량 알림 설정
- **접근 감사**: PVC 생성/삭제 이벤트 감사 로그 수집

---

## 참고 자료

- Kubernetes PV/PVC 공식 문서: <https://kubernetes.io/docs/concepts/storage/persistent-volumes/>
- 관련 문서: StorageClass, StatefulSet