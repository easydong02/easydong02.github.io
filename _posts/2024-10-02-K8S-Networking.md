---
title: "[Kubernetes] Role, Clusterrole, Binding"
date: 2024-10-02 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, rbac, role, clusterrole, binding, network]
render_with_liquid: false
---

# Kubernetes 기본 보안 구성

Kubernetes는 컨테이너 오케스트레이션 시스템으로, 클러스터 환경에서 보안은 매우 중요한 요소입니다. Kubernetes는 다양한 보안 메커니즘을 제공하여 클러스터와 애플리케이션을 보호하고 관리할 수 있습니다. 이번 포스팅에서는 Kubernetes의 보안 구성 중에서 **RBAC(Role-Based Access Control)**, **ServiceAccount**, **NetworkPolicy**를 통해 보안을 강화하는 방법을 살펴보겠습니다.

## 1. Role-Based Access Control (RBAC)

**RBAC**는 Kubernetes에서 사용자, 그룹, 애플리케이션 또는 서비스가 수행할 수 있는 작업을 제어하는 메커니즘입니다. **Role**과 **ClusterRole**은 리소스에 대한 권한을 정의하며, **RoleBinding**과 **ClusterRoleBinding**은 이러한 역할을 사용자에게 부여합니다.

### 1.1 Role과 RoleBinding

**Role**은 특정 네임스페이스 내에서만 유효한 권한을 정의하며, **RoleBinding**을 통해 해당 Role을 사용자 또는 서비스 계정에 연결할 수 있습니다.

### Role 예시

```yaml
yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-ns
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

위 예시는 `dev-ns` 네임스페이스에서 Pod를 조회하는 권한을 가진 Role을 정의한 것입니다.

### RoleBinding 예시

```yaml
yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev-ns
subjects:
- kind: User
  name: "dev-user"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

이 예시는 `dev-user`에게 `pod-reader` Role을 부여하여, `dev-ns` 네임스페이스에서 Pod를 조회할 수 있는 권한을 부여합니다.

### 1.2 ClusterRole과 ClusterRoleBinding

**ClusterRole**은 클러스터 전체에서 유효한 권한을 정의합니다. **ClusterRoleBinding**을 통해 해당 ClusterRole을 특정 사용자나 그룹에 할당할 수 있습니다.

### ClusterRole 예시

```yaml
yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "delete"]
```

### ClusterRoleBinding 예시

```yaml
yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
- kind: User
  name: "admin-user"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin-role
  apiGroup: rbac.authorization.k8s.io
```

위의 설정을 통해 `admin-user`는 클러스터 전체에서 Pod를 조회, 삭제할 수 있는 권한을 얻게 됩니다.

## 2. ServiceAccount

**ServiceAccount**는 Kubernetes에서 Pod와 같은 내부 애플리케이션이 API 서버와 상호작용할 때 사용하는 계정입니다. 각 Pod는 기본적으로 ServiceAccount를 통해 권한을 얻어 작업을 수행합니다.

### ServiceAccount 생성 예시

```bash
kubectl create serviceaccount dev-sa -n dev-ns
```

이 명령어는 `dev-ns` 네임스페이스에 `dev-sa`라는 ServiceAccount를 생성합니다.

### ServiceAccount를 사용하는 Pod 예시

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
  namespace: dev-ns
spec:
  serviceAccountName: dev-sa
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

이 설정은 `dev-sa` ServiceAccount를 사용하는 Pod를 정의합니다. 해당 Pod는 `dev-sa` 계정을 통해 Kubernetes API 서버와 상호작용할 수 있습니다.

## 3. NetworkPolicy

**NetworkPolicy**는 Kubernetes에서 네트워크 트래픽을 제어하는 정책을 정의하는 오브젝트입니다. 이를 통해 Pod 간의 통신을 제어하고, 외부 네트워크로부터의 접근을 제한할 수 있습니다.

### 기본 NetworkPolicy 예시

다음은 특정 Pod에 대한 모든 외부 트래픽을 차단하는 NetworkPolicy 예시입니다.

```yaml
yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-traffic
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress: []
  egress: []
```

이 설정은 `myapp` 라벨이 붙은 모든 Pod로 들어오는 트래픽과 나가는 트래픽을 차단합니다.

### 특정 트래픽 허용 예시

특정 트래픽만 허용하려면 다음과 같은 정책을 적용할 수 있습니다.

```yaml
yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-traffic
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
    ports:
    - protocol: TCP
      port: 80
```

이 설정은 `myapp` 라벨이 있는 Pod에 192.168.1.0/24 네트워크 대역에서 오는 TCP 80번 포트의 트래픽만 허용합니다.

## 4. Kubernetes Dashboard 보안 구성

**Kubernetes Dashboard**는 클러스터를 시각적으로 관리할 수 있는 UI입니다. 그러나, 클러스터 전체를 관리할 수 있는 권한을 부여받기 위해서는 적절한 권한 설정이 필요합니다.

### Dashboard 접근을 위한 ServiceAccount 생성

```yaml
yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

### ClusterRoleBinding 설정

```yaml
yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

이 설정을 통해 `admin-user`라는 ServiceAccount는 클러스터 관리자 권한을 부여받게 됩니다.