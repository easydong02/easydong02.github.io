---
title: "[CI/CD] HyperCloud CICD Operator + Tekton Pipeline Controller 완전 정리"
date: 2026-02-23 00:00:00 +0900
categories: [Infra, CICD]
tags: [kubernetes, hypercloud, cicd, tekton, operator, pipelinerun, integrationconfig, gitlab]
render_with_liquid: false
---

# HyperCloud CICD Operator

## 1. 개요

**한 줄 요약**: IntegrationConfig 리소스를 감시하여 Git 이벤트 발생 시 Tekton PipelineRun을 자동으로 생성하는 Kubernetes Operator

**핵심 역할**:
- GitLab Webhook 자동 등록 및 이벤트 수신
- IntegrationConfig → PipelineRun 자동 변환
- Secret, ConfigMap, podTemplate 자동 주입
- CI/CD 파이프라인 실행 자동화

**비유**:

```
IntegrationConfig = 파이프라인 설계도
CICD Operator     = 설계도를 보고 자동으로 공장(PipelineRun)을 가동시키는 관리자
```

---

## 2. 아키텍처

### 계층 구조

```
┌──────────────────────────────────────────┐
│ 사용자 작성                                 │
├──────────────────────────────────────────┤
│ IntegrationConfig                        │
│ ├─ git: repository, branch               │
│ ├─ jobs: [build, deploy]                 │
│ ├─ secrets: [harbor-secret]              │
│ └─ podTemplate: nodeSelector             │
│                                          │
│ Task                                     │
│ └─ steps, params, volumes                │
└──────────────────────────────────────────┘
              ↓ Git Push/Tag 이벤트
┌──────────────────────────────────────────┐
│ CICD Operator (자동 생성)                  │
├──────────────────────────────────────────┤
│ PipelineRun                              │
│ ├─ tasks: IC의 jobs 배열 변환               │
│ ├─ podTemplate: IC 설정 상속               │
│ ├─ secrets: 자동 주입                      │
│ └─ 환경변수: CI_*, GIT_* 자동 추가           │
└──────────────────────────────────────────┘
              ↓
┌──────────────────────────────────────────┐
│ Tekton Pipeline Controller (자동 생성)     │
├──────────────────────────────────────────┤
│ TaskRun (각 Job마다)                       │
│ └─ PipelineRun 설정 상속                   │
└──────────────────────────────────────────┘
              ↓
┌──────────────────────────────────────────┐
│ Tekton TaskRun Controller (자동 생성)      │
├──────────────────────────────────────────┤
│ Pod                                      │
│ ├─ Task steps → Container 변환            │
│ ├─ Secret/ConfigMap 마운트                 │
│ └─ nodeSelector, tolerations 적용         │
└──────────────────────────────────────────┘
```

### 리소스 관계 (OwnerReference)

```
IntegrationConfig (IC05)
  └─> PipelineRun (aic-src-lata-shs-ic05-run-xxxxx)
       └─> TaskRun (build-image)
            └─> Pod (build-image-pod)

삭제 시 자동 cascade:
IC 삭제 → PipelineRun 삭제 → TaskRun 삭제 → Pod 삭제
```

---

## 3. 실행 흐름 (단계별)

### 1단계: IntegrationConfig 생성

```yaml
apiVersion: cicd.tmax.io/v1
kind: IntegrationConfig
metadata:
  name: aic-src-lata-shs-ic05
  namespace: shg-cicd-aic
spec:
  git:
    repository: aic/src/aic-src-lata-shs
    apiUrl: https://gitlab.example.com/api/v4

  jobs:
    postSubmit:  # Tag Push 이벤트 시
      - name: pull-package
        tektonTask:
          taskRef:
            name: pull-package
      - name: build-image
        tektonTask:
          taskRef:
            name: build-image-prd
        when:
          branch: (0|1|2|3|4|5|6|7|8|9)*  # 숫자로 시작하는 태그만

  secrets:
    - name: shg-harbor-cicdmgr-sec-aic

  podTemplate:
    nodeSelector:
      node: aicmgp
    tolerations:
      - key: node
        value: aicmgp
        effect: NoSchedule
```

```bash
kubectl apply -f ic05.yaml
```

### 2단계: CICD Operator가 Webhook 등록

```
CICD Operator 동작:
1. IntegrationConfig 생성 감지 (Kubernetes Watch)
2. git.repository 정보 추출
3. GitLab API 호출 → Webhook 등록
   URL: http://cicd-webhook.shg-cicd-aic.svc:8080/webhooks
   Events: push, tag_push, merge_request
```

### 3단계: Git 이벤트 발생

```bash
# 개발자가 태그 생성
git tag 1.0.0
git push origin 1.0.0
```

**GitLab → CICD Operator**:

```json
POST http://cicd-webhook.shg-cicd-aic.svc:8080/webhooks
{
  "object_kind": "tag_push",
  "ref": "refs/tags/1.0.0",
  "repository": {
    "name": "aic-src-lata-shs"
  }
}
```

### 4단계: PipelineRun 자동 생성

CICD Operator가 생성하는 PipelineRun:

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: aic-src-lata-shs-ic05-run-abc123
  namespace: shg-cicd-aic
  labels:
    cicd.tmax.io/integration-config: aic-src-lata-shs-ic05
  ownerReferences:
    - apiVersion: cicd.tmax.io/v1
      kind: IntegrationConfig
      name: aic-src-lata-shs-ic05
spec:
  pipelineSpec:
    tasks:
      - name: pull-package
        taskRef:
          name: pull-package
      - name: build-image
        taskRef:
          name: build-image-prd
        runAfter: [pull-package]

  # IntegrationConfig의 podTemplate 자동 주입
  podTemplate:
    nodeSelector:
      node: aicmgp
    tolerations:
      - key: node
        value: aicmgp
        effect: NoSchedule
    imagePullSecrets:
      - name: shg-harbor-cicdmgr-sec-aic
    volumes:
      - name: docker-config
        secret:
          secretName: shg-harbor-cicdmgr-sec-aic

  # 환경 변수 자동 주입
  params:
    - name: CI_SERVER_URL
      value: "https://gitlab.example.com"
    - name: CI_REPOSITORY
      value: "aic/src/aic-src-lata-shs"
    - name: CI_HEAD_REF
      value: "1.0.0"
    - name: CI_COMMIT_SHA
      value: "abc123..."
```

### 5단계: Tekton이 TaskRun → Pod 생성

```
Tekton Pipeline Controller:
  PipelineRun → TaskRun (build-image) 생성

Tekton TaskRun Controller:
  TaskRun → Pod 생성
    ├─ Task의 steps를 Container로 변환
    ├─ PipelineRun의 podTemplate 상속
    ├─ Secret을 Volume으로 마운트
    └─ Tekton 내부 Volume 자동 추가
```

최종 생성되는 Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ic05-run-abc123-build-image-pod
spec:
  nodeSelector:
    node: aicmgp
  tolerations:
    - key: node
      value: aicmgp
  imagePullSecrets:
    - name: shg-harbor-cicdmgr-sec-aic
  containers:
  - name: step-build
    image: hyperregistry.aic.shinhan.com/.../podman:stable
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
        readOnly: true
  volumes:
    - name: docker-config
      secret:
        secretName: shg-harbor-cicdmgr-sec-aic
```

---

## 4. 주요 기능

### 4.1. Git Webhook 자동 관리

```yaml
spec:
  git:
    repository: myapp
    apiUrl: https://gitlab.example.com/api/v4
```

- GitLab API 호출하여 Webhook 자동 등록
- IC 삭제 시 Webhook 자동 해제
- 이벤트 필터링 (push, tag_push, merge_request)

### 4.2. 조건부 실행 (when)

```yaml
jobs:
  preSubmit:   # Merge Request 시 실행
    - name: test
      when:
        branch: (dev|stg).*      # dev, stg로 시작하는 브랜치만

  postSubmit:  # Tag Push 시 실행
    - name: deploy
      when:
        branch: (0|1|2|3|4|5|6|7|8|9)*  # 숫자로 시작하는 태그만
```

### 4.3. Secret 자동 주입

```yaml
spec:
  secrets:
    - name: harbor-secret
    - name: kubeconfig-secret
```

- PipelineRun의 `podTemplate.imagePullSecrets`에 자동 추가
- Volume으로 마운트 (`/kaniko/.docker/config.json`)

### 4.4. 환경 변수 자동 주입

CICD Operator가 PipelineRun에 자동으로 추가하는 환경 변수:

| 변수 | 값 예시 | 설명 |
|------|---------|------|
| `CI_SERVER_URL` | https://gitlab.example.com | GitLab 서버 URL |
| `CI_REPOSITORY` | aic/src/myapp | 저장소 경로 |
| `CI_HEAD_REF` | 1.0.0 | 브랜치/태그 이름 |
| `CI_COMMIT_SHA` | abc123... | Commit SHA |
| `CI_CONFIG_NAME` | ic05 | IntegrationConfig 이름 |

---

## 5. CICD Operator vs 수동 방식 비교

| 항목 | CICD Operator | 수동 PipelineRun |
|------|--------------|------------------|
| **PipelineRun 생성** | 자동 (Git 이벤트 시) | 수동 (`kubectl apply`) |
| **Webhook 관리** | 자동 등록/해제 | 수동 설정 |
| **Secret 주입** | 자동 (IC의 secrets) | 수동 작성 |
| **환경 변수** | 자동 주입 (CI_*, GIT_*) | 수동 작성 |
| **조건부 실행** | when 조건 자동 평가 | 불가능 |
| **재사용성** | IC 하나로 무한 재사용 | 매번 새로 작성 |

---

## 6. 확인 명령어

### IntegrationConfig 확인

```bash
# IC 목록
kubectl get integrationconfig -n shg-cicd-aic
kubectl get ic -n shg-cicd-aic

# 상세 정보
kubectl describe ic aic-src-lata-shs-ic05 -n shg-cicd-aic
```

### PipelineRun 확인

```bash
# 특정 IC의 PipelineRun
kubectl get pipelinerun -n shg-cicd-aic \
  -l cicd.tmax.io/integration-config=aic-src-lata-shs-ic05

# 최근 실행 5개
kubectl get pr -n shg-cicd-aic --sort-by=.metadata.creationTimestamp | tail -5

# 상세 정보
kubectl describe pr <pipelinerun-name> -n shg-cicd-aic
```

### TaskRun 확인

```bash
# 특정 PipelineRun의 TaskRun
kubectl get taskrun -n shg-cicd-aic \
  -l tekton.dev/pipelineRun=<pipelinerun-name>

# build-image TaskRun만
kubectl get tr -n shg-cicd-aic | grep build-image
```

### Pod 확인

```bash
# 특정 PipelineRun의 Pod
kubectl get pod -n shg-cicd-aic \
  -l tekton.dev/pipelineRun=<pipelinerun-name>

# Pod의 Volume 확인
kubectl describe pod <pod-name> -n shg-cicd-aic | grep -A 10 Volumes
```

### 전체 확인 스크립트

```bash
#!/bin/bash
NAMESPACE="shg-cicd-aic"
IC_NAME="aic-src-lata-shs-ic05"

echo "=== IntegrationConfig ==="
kubectl get ic $IC_NAME -n $NAMESPACE

echo -e "\n=== 최근 PipelineRun ==="
kubectl get pr -n $NAMESPACE \
  -l cicd.tmax.io/integration-config=$IC_NAME \
  --sort-by=.metadata.creationTimestamp | tail -3

LATEST_PR=$(kubectl get pr -n $NAMESPACE \
  -l cicd.tmax.io/integration-config=$IC_NAME \
  --sort-by=.metadata.creationTimestamp -o name | tail -1)

if [ ! -z "$LATEST_PR" ]; then
  echo -e "\n=== TaskRun (${LATEST_PR##*/}) ==="
  kubectl get tr -n $NAMESPACE -l tekton.dev/pipelineRun=${LATEST_PR##*/}

  echo -e "\n=== Pod ==="
  kubectl get pod -n $NAMESPACE -l tekton.dev/pipelineRun=${LATEST_PR##*/} -o wide
fi
```

---

## 7. 트러블슈팅

### Webhook 이벤트를 받았지만 PipelineRun 생성 안 됨

```bash
# when 조건 확인 (브랜치/태그 패턴 불일치 가능)
kubectl get ic <ic-name> -n <namespace> -o yaml | grep -A 3 when
```

### Secret 마운트 실패

```bash
# Secret 존재 확인
kubectl get secret shg-harbor-cicdmgr-sec-aic -n shg-cicd-aic

# IC에 secrets 배열 추가 확인
kubectl get ic <ic-name> -n <namespace> -o yaml | grep -A 3 secrets
```

### Pod가 다른 노드에 스케줄링됨

```bash
# podTemplate 확인
kubectl get ic <ic-name> -n <namespace> -o yaml | grep -A 5 podTemplate

# PipelineRun에 반영되었는지 확인
kubectl get pr <pr-name> -n <namespace> -o yaml | grep -A 5 podTemplate
```

### 환경 변수 누락

```bash
# Pod 환경 변수 확인
kubectl exec <pod-name> -n <namespace> -- env | grep CI_

# CICD Operator 로그 확인
kubectl logs -n cicd-system deploy/cicd-operator | grep -i error
```

### 디버깅 로그 확인

```bash
# CICD Operator 로그
kubectl logs -n cicd-system deploy/cicd-operator -f

# Webhook 수신 로그
kubectl logs -n cicd-system deploy/cicd-webhook -f

# IC 이벤트 확인
kubectl get events -n shg-cicd-aic --field-selector involvedObject.name=<ic-name>

# PipelineRun 실패 원인
kubectl describe pr <pr-name> -n shg-cicd-aic | grep -A 10 "Message:"
```

---

## 8. 요약

**CICD Operator의 역할**:

```
✅ GitLab Webhook 자동 등록
✅ Git 이벤트 수신 및 필터링
✅ IntegrationConfig → PipelineRun 변환
✅ Secret, ConfigMap 자동 주입
✅ podTemplate 전파
✅ 환경 변수 자동 추가
```

**사용자의 역할**:

```
✅ IntegrationConfig 작성 (한 번만)
✅ Task 작성 (재사용)
✅ Secret 생성 (레지스트리 인증)
```

**자동화 전체 흐름**:

```
Git Push/Tag
  ↓
GitLab Webhook
  ↓
CICD Operator (PipelineRun 생성)
  ↓
Tekton Pipeline Controller (TaskRun 생성)
  ↓
Tekton TaskRun Controller (Pod 생성)
  ↓
Kubernetes (Container 실행)
```

---

## 참고

- HyperCloud CICD 공식 문서: <https://docs.hypercloud.com/cicd/>
- **HyperCloud 전용** Operator (Tmax 자체 개발), 표준 오픈소스 대안은 ArgoCD Events + Tekton Triggers