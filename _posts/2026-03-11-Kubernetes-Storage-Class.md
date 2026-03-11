---
title: "[Kubernetes] StorageClass"
date: 2026-03-11 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, storageclass, pv, pvc, storage]
render_with_liquid: false
---

## 📌 개요

**한 줄 요약**: 동적 볼륨 프로비저닝을 위한 스토리지 템플릿 리소스

**언제 사용**:
- PVC 생성 시 자동으로 PV를 프로비저닝하고 싶을 때
- 여러 스토리지 타입(EFS, EBS, Local Path 등)을 구분해서 사용할 때
- Tekton Pipeline에서 Workspace 볼륨을 동적으로 생성할 때

**관련 기술**: PersistentVolume, PersistentVolumeClaim, CSI Driver, Tekton Workspace

---

## 🎯 핵심 개념

### Static vs Dynamic Provisioning

| 항목 | Static Provisioning | Dynamic Provisioning |
|------|---------------------|----------------------|
| **PV 생성** | 관리자가 수동 생성 | StorageClass가 자동 생성 |
| **사용 시기** | 레거시, 특수 스토리지 | 클라우드, 일반적 사용 |
| **장점** | 세밀한 제어 | 자동화, 편리 |
| **단점** | 수동 관리 부담 | StorageClass 의존 |

```
[Static]
관리자가 PV 미리 생성 → PVC가 기존 PV를 찾아서 바인딩

[Dynamic]
PVC 생성 → StorageClass가 PV 자동 생성 및 바인딩
```

### Provisioner

실제 스토리지를 생성하는 주체로, 각 Provisioner마다 고유한 `parameters` 설정이 필요합니다.

| Provisioner | 스토리지 종류 |
|------------|-------------|
| `rancher.io/local-path` | 노드 로컬 디스크 |
| `efs.csi.aws.com` | AWS EFS (공유 파일 시스템) |
| `ebs.csi.aws.com` | AWS EBS (블록 스토리지) |

### VolumeBindingMode

| 모드 | 동작 | 사용 시기 |
|------|------|-----------|
| **Immediate** | PVC 생성 즉시 PV 프로비저닝 | EFS, NFS (노드 위치 무관) |
| **WaitForFirstConsumer** | Pod 스케줄링 시점에 PV 생성 | Local Path, EBS (노드 위치 중요) |

### Reclaim Policy

| 정책 | 동작 | 사용 시기 |
|------|------|-----------|
| **Delete** | PVC 삭제 시 PV와 실제 스토리지 함께 삭제 | 개발/테스트 |
| **Retain** | PVC 삭제해도 PV 유지, 데이터 보존 | 프로덕션 |

---

## 💻 기본 명령어

```bash
# StorageClass 목록 조회
kubectl get sc

# StorageClass 상세 확인
kubectl describe sc <sc-name>

# 특정 StorageClass를 사용하는 PVC 조회
kubectl get pvc -A | grep <sc-name>
```

**출력 예시**:
```
NAME                PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION
efs-sc              efs.csi.aws.com          Delete          Immediate              false
local-path(default) rancher.io/local-path    Delete          WaitForFirstConsumer   false
```

---

## 🔧 자주 쓰는 패턴

### 패턴 1: local-path (기본 StorageClass)

HyperCloud/Rancher 환경의 기본 StorageClass입니다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

**특징**:
- `WaitForFirstConsumer`: Pod가 스케줄링될 노드에서 볼륨 생성
- PVC만 생성하면 `Pending` 상태 → Pod 생성 시 PV 자동 생성
- 노드 간 데이터 공유 불가 (Local Storage)

```
PVC 생성 → Pending
    ↓
Pod 스케줄링 (노드 결정)
    ↓
해당 노드에 PV 자동 생성 → Bound
    ↓
Pod 기동
```

### 패턴 2: efs-sc (공유 스토리지)

여러 Pod/노드에서 동시에 접근해야 할 때 사용합니다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

**특징**:
- `Immediate`: PVC 생성 즉시 PV 프로비저닝
- 여러 Pod/노드에서 동시 `ReadWriteMany` 가능
- Tekton Pipeline Workspace 공유에 적합

### 패턴 3: Tekton Pipeline Workspace에서 사용

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-pipeline-run
spec:
  pipelineRef:
    name: build-pipeline
  workspaces:
  - name: shared-workspace
    volumeClaimTemplate:
      spec:
        accessModes:
        - ReadWriteOnce
        storageClassName: local-path   # StorageClass 지정
        resources:
          requests:
            storage: 1Gi
```

**Tekton 사용 시 주의사항**:
- `volumeClaimTemplate`: PipelineRun마다 새로운 PVC 자동 생성
- `local-path` (WaitForFirstConsumer): Task Pod가 스케줄링된 후 볼륨 생성
- Workspace 경로: `$(workspaces.<name>.path)` → `/workspace/<name>`

---

## ⚠️ 주의사항

### volumeBindingMode 혼동

```
❌ local-path를 Immediate로 설정
→ PVC 생성 시 특정 노드 선택 불가로 실패

✅ local-path는 WaitForFirstConsumer 유지
→ Pod 스케줄링 후 해당 노드에 볼륨 생성
```

### default StorageClass 중복 설정

```bash
# default StorageClass 확인
kubectl get sc

# (default) 표시가 2개 이상이면 PVC 바인딩 혼란 발생
# → 하나만 default로 유지

# default 해제
kubectl patch sc <sc-name> \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

### PVC Pending 상태 디버깅

```bash
# 원인 확인
kubectl describe pvc <pvc-name> -n <namespace>

# 주요 원인:
# - WaitForFirstConsumer → Pod 아직 생성 안 됨 (정상)
# - 조건 맞는 PV 없음
# - StorageClass 이름 오타
# - 노드 디스크 용량 부족
```

---

## 📝 요약

```
StorageClass:
- 동적 볼륨 프로비저닝 템플릿
- Provisioner가 실제 스토리지 생성

핵심 필드:
- provisioner: 스토리지 생성 주체
- reclaimPolicy: PVC 삭제 시 PV 처리 방식
- volumeBindingMode: PV 생성 시점 (즉시 or Pod 스케줄링 후)

환경별 선택:
- local-path: 단일 노드, 빠른 I/O (Tekton 컴파일 등)
- efs-sc: 다중 노드 공유 (ReadWriteMany 필요 시)
```

---

## 🔗 관련 문서

- [공식 문서 - Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)