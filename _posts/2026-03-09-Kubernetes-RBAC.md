---
title: "[Kubernetes] RBAC 완전 정리: 역할 기반 접근 제어 메커니즘과 실무 패턴"
date: 2026-03-09 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, rbac, role, clusterrole, rolebinding, serviceaccount, security, authorization]
render_with_liquid: false
---

# Kubernetes RBAC (Role-Based Access Control)

## 1. 개요

**한 줄 요약**: 누가(Subject) 무엇을(Resource) 어떻게(Verb) 할 수 있는지 정의하는 역할 기반 접근 제어 시스템

**핵심 구성 요소**:

```
1. Role / ClusterRole          : 권한 정의 (무엇을 할 수 있는지)
2. RoleBinding / ClusterRoleBinding : 권한 부여 (누구에게)
3. Subject                     : 사용자 / 그룹 / ServiceAccount
```

**권한 관리 원칙**:
- **Allow Only**: 허용만 있고 거부(deny) 없음
- **합집합(Union)**: 여러 Binding의 권한은 누적됨
- **최소 권한**: 필요한 만큼만 부여

---

## 2. Role vs ClusterRole

### 2.1. 기본 차이점

| 항목 | Role | ClusterRole |
|------|------|-------------|
| **범위** | 특정 네임스페이스 | 클러스터 전체 |
| **네임스페이스 리소스** | ✅ Pod, Service 등 | ✅ Pod, Service 등 |
| **클러스터 리소스** | ❌ Node, PV 불가 | ✅ Node, PV 가능 |
| **재사용성** | ❌ 한 네임스페이스만 | ✅ 여러 곳에서 재사용 |
| **metadata.namespace** | 필수 | 없음 |

### 2.2. Role (네임스페이스 범위)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default   # 특정 네임스페이스에만 적용
rules:
- apiGroups: [""]      # core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### 2.3. ClusterRole (클러스터 전체)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader    # namespace 없음
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### 2.4. 리소스 타입별 분류

**네임스페이스 리소스** (Role 가능):

```
pods, services, deployments, replicasets, configmaps,
secrets, persistentvolumeclaims, ingresses, jobs, cronjobs
```

**클러스터 리소스** (ClusterRole만 가능):

```
nodes, persistentvolumes, namespaces, storageclasses,
clusterroles, clusterrolebindings
```

---

## 3. RoleBinding vs ClusterRoleBinding

### 3.1. Binding 조합 4가지

#### 패턴 1: Role + RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: donghee
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

결과: default 네임스페이스의 Pod만 조회 가능. 특정 네임스페이스만 권한이 필요할 때 사용합니다.

#### 패턴 2: ClusterRole + ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes
subjects:
- kind: User
  name: donghee
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

결과: 클러스터 전체의 모든 Node 조회 가능. 클러스터 전체 권한이 필요할 때 사용합니다.

#### 패턴 3: ClusterRole + RoleBinding ⭐ 실무 추천

ClusterRole을 한 번 정의하고, 여러 네임스페이스의 RoleBinding에서 재사용하는 방식입니다.

```yaml
# ClusterRole (재사용 가능하게 정의)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-admin
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]
---
# RoleBinding 1 (dev 네임스페이스)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-pod-admin
  namespace: dev
subjects:
- kind: User
  name: developer
roleRef:
  kind: ClusterRole    # ClusterRole 참조
  name: pod-admin
  apiGroup: rbac.authorization.k8s.io
---
# RoleBinding 2 (staging 네임스페이스)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: staging-pod-admin
  namespace: staging
subjects:
- kind: User
  name: developer
roleRef:
  kind: ClusterRole    # 동일 ClusterRole 재사용
  name: pod-admin
  apiGroup: rbac.authorization.k8s.io
```

결과: dev, staging 네임스페이스 → Pod 전체 권한 / production 네임스페이스 → 권한 없음

#### 패턴 4: Role + ClusterRoleBinding ❌ 불가능

```yaml
# 에러 발생
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
roleRef:
  kind: Role    # ❌ ClusterRoleBinding can only reference ClusterRole
  name: pod-reader
```

### 3.2. Binding 조합 요약표

| Binding 타입 | 참조 가능 | 권한 범위 |
|-------------|---------|---------|
| **RoleBinding** | Role ✅ | 해당 네임스페이스만 |
| **RoleBinding** | ClusterRole ✅ | 해당 네임스페이스만 |
| **ClusterRoleBinding** | ClusterRole ✅ | 클러스터 전체 |
| **ClusterRoleBinding** | Role ❌ | 불가능 |

---

## 4. 권한 합산: 합집합(Union) 원칙

### 4.1. 핵심 원칙

```
Kubernetes RBAC 특징:
1. 권한은 누적됨 (합집합)
2. deny(거부) 개념 없음
3. 한 번이라도 허용되면 가능

여러 RoleBinding이 있으면 = 모든 권한의 합!
```

deny가 없는 이유는 허용과 거부가 동시에 존재하면 충돌이 발생하기 때문입니다. Kubernetes는 allow만 허용하는 단순한 모델을 채택했으며, 이 때문에 **최소 권한 원칙**이 더욱 중요합니다.

### 4.2. 합산 예시: 같은 네임스페이스, 다른 리소스

```yaml
# RoleBinding 1: Pod 읽기
subjects:
- kind: User
  name: donghee
roleRef:
  name: pod-reader    # get, list, watch pods
---
# RoleBinding 2: Service 편집
subjects:
- kind: User
  name: donghee
roleRef:
  name: service-editor  # get, list, create, update, delete services
```

최종 권한:

```
donghee @ default 네임스페이스:
✅ Pods:     get, list, watch
✅ Services: get, list, create, update, delete
```

### 4.3. 합산 예시: ClusterRoleBinding + RoleBinding 혼합

```yaml
# ClusterRoleBinding: 클러스터 전체 읽기
roleRef:
  kind: ClusterRole
  name: view     # 모든 네임스페이스 읽기
---
# RoleBinding: default 네임스페이스 편집
metadata:
  namespace: default
roleRef:
  kind: ClusterRole
  name: edit     # default 네임스페이스 편집
```

최종 권한:

```
default 네임스페이스:
✅ 모든 리소스 읽기  (ClusterRoleBinding)
✅ 대부분 리소스 편집 (RoleBinding)
→ 편집 권한 (더 강한 권한 적용)

다른 모든 네임스페이스:
✅ 모든 리소스 읽기만 (ClusterRoleBinding)
```

---

## 5. 내장 ClusterRole

| ClusterRole | 권한 범위 | 용도 |
|-------------|-----------|------|
| **view** | 대부분 리소스 읽기 (Secret 제외) | 개발자, 모니터링 |
| **edit** | view + 대부분 리소스 생성/수정/삭제 | 개발자 (배포 포함) |
| **admin** | edit + Role/RoleBinding 관리 | 네임스페이스 관리자 |
| **cluster-admin** | 모든 권한 (클러스터 포함) | 클러스터 관리자 ⚠️ |

---

## 6. 기본 명령어

```bash
# 자신의 모든 권한 나열
kubectl auth can-i --list
kubectl auth can-i --list -n default

# 특정 작업 가능 여부
kubectl auth can-i create pods
kubectl auth can-i delete pods -n production
kubectl auth can-i get nodes
kubectl auth can-i '*' '*'   # 모든 권한 여부

# 다른 사용자 권한 확인 (관리자용)
kubectl auth can-i get pods --as=donghee
kubectl auth can-i --list --as=donghee -n default

# ServiceAccount 권한 확인
kubectl auth can-i get pods \
  --as=system:serviceaccount:default:my-sa
```

### RoleBinding 탐색

```bash
# 특정 사용자의 모든 RoleBinding
kubectl get rolebindings -A -o json | \
  jq -r '.items[] | select(.subjects[]?.name == "donghee") |
  "\(.metadata.namespace)/\(.metadata.name) -> \(.roleRef.name)"'

# ClusterRoleBinding 탐색
kubectl get clusterrolebindings -o json | \
  jq -r '.items[] | select(.subjects[]?.name == "donghee") |
  "\(.metadata.name) -> \(.roleRef.name)"'
```

### 위험한 권한 감사

```bash
# cluster-admin 권한 보유자 목록
kubectl get clusterrolebindings -o json | \
  jq -r '.items[] | select(.roleRef.name == "cluster-admin") |
  "⚠️  \(.subjects[].name) has cluster-admin"'

# 네임스페이스 admin 권한 보유자 목록
kubectl get rolebindings -A -o json | \
  jq -r '.items[] | select(.roleRef.name == "admin") |
  "⚠️  \(.subjects[].name) has admin in \(.metadata.namespace)"'
```

---

## 7. 실무 패턴

### 7.1. 개발자 권한 (환경별 분리)

```yaml
# dev: 모든 권한
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-dev-admin
  namespace: dev
subjects:
- kind: User
  name: developer1
roleRef:
  kind: ClusterRole
  name: admin
---
# staging: 배포 및 조회
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-staging-edit
  namespace: staging
subjects:
- kind: User
  name: developer1
roleRef:
  kind: ClusterRole
  name: edit
---
# production: 조회만
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-prod-view
  namespace: production
subjects:
- kind: User
  name: developer1
roleRef:
  kind: ClusterRole
  name: view
```

### 7.2. DevOps 엔지니어

```yaml
# 클러스터 전체 조회
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops-cluster-viewer
subjects:
- kind: User
  name: devops1
roleRef:
  kind: ClusterRole
  name: view
---
# Node 관리 (별도 ClusterRole 정의 후 바인딩)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops-node-reader
subjects:
- kind: User
  name: devops1
roleRef:
  kind: ClusterRole
  name: node-reader
---
# kube-system 관리
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devops-kube-system-admin
  namespace: kube-system
subjects:
- kind: User
  name: devops1
roleRef:
  kind: ClusterRole
  name: admin
```

### 7.3. CI/CD 파이프라인 (GitLab Runner)

```yaml
# ClusterRole 정의 (재사용)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
# dev 배포 허용
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-cd-dev-deployer
  namespace: dev
subjects:
- kind: ServiceAccount
  name: gitlab-runner
  namespace: ci-cd
roleRef:
  kind: ClusterRole
  name: deployer
---
# staging 배포 허용
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: ci-cd-staging-deployer
  namespace: staging
subjects:
- kind: ServiceAccount
  name: gitlab-runner
  namespace: ci-cd
roleRef:
  kind: ClusterRole
  name: deployer

# production은 RoleBinding 없음 → 배포 불가 ✅
```

---

## 8. 트러블슈팅

### 8.1. 권한 거부 에러

```bash
# 에러 메시지
Error from server (Forbidden): pods is forbidden:
User "donghee" cannot list resource "pods"

# 1. 권한 확인
kubectl auth can-i list pods --as=donghee -n default
# no

# 2. RoleBinding 확인
kubectl get rolebindings -n default -o yaml | grep -A 10 donghee

# 3. 해결: RoleBinding 생성
kubectl create rolebinding donghee-pod-reader \
  --clusterrole=view \
  --user=donghee \
  -n default
```

### 8.2. RoleBinding 작동 안 함 체크리스트

```
1. Subject 이름 확인 (대소문자, 철자 오타)
2. Namespace 확인 (RoleBinding과 동일한지)
3. RoleRef 확인 (Role 이름, 실제 존재 여부)
4. Role 내용 확인 (필요한 verb, apiGroup)
```

---

## 9. 주의사항 및 권장사항

### 9.1. 최소 권한 원칙

```yaml
# ❌ 나쁜 예
kind: ClusterRoleBinding
roleRef:
  name: cluster-admin   # 너무 강함!

# ✅ 좋은 예
kind: RoleBinding
namespace: dev
roleRef:
  name: edit            # 필요한 것만
```

### 9.2. ClusterRole 재사용 패턴

```yaml
# ✅ ClusterRole 한 번 정의
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]
---
# RoleBinding으로 여러 네임스페이스에 재사용
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-pod-manager
  namespace: dev
roleRef:
  kind: ClusterRole    # 재사용
  name: pod-manager
```

### 9.3. ServiceAccount 사용 권장

```yaml
# ❌ 사용자 계정 직접 사용
subjects:
- kind: User
  name: jenkins

# ✅ ServiceAccount 사용 (권장)
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: ci-cd
```

### 9.4. 권장 패턴 우선순위

```
⭐⭐⭐ ClusterRole + RoleBinding  (실무 최우선 권장)
     - ClusterRole 한 번 정의, 여러 네임스페이스에 재사용
     - 권한 관리 간편

⭐⭐  ClusterRole + ClusterRoleBinding
     - 클러스터 전체 권한이 진짜 필요한 경우에만

⭐   Role + RoleBinding
     - 단일 네임스페이스에서만 사용하는 경우

보안 원칙:
- 최소 권한 부여
- 정기적 권한 감사
- ServiceAccount 사용
- cluster-admin 최소화
```

---

## 참고 자료

- Kubernetes RBAC 공식 문서: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>