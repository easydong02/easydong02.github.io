---
title: "[Kubernetes] 네임 스페이스(Namespace)"
date: 2024-09-30 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, namespace, ns]
render_with_liquid: false
---

# Kubernetes Namespace 관리

Kubernetes에서 **Namespace**는 클러스터 내의 리소스를 분리하고 조직화하는 중요한 방법입니다. 여러 팀이 하나의 Kubernetes 클러스터를 공유할 때, 네트워크, 리소스, 권한을 격리하여 효율적으로 관리할 수 있는 방법을 제공하는 것이 바로 Namespace입니다. 이번 포스팅에서는 Namespace의 개념, 생성 방법, ResourceQuota 및 LimitRange 설정에 대해 자세히 알아보겠습니다.

## 1. Namespace의 개념

Namespace는 Kubernetes에서 리소스를 논리적으로 분리하는 단위입니다. 기본적으로 생성되는 네임스페이스는 다음과 같습니다:

- **default**: 기본 Namespace로, 명시적으로 네임스페이스를 지정하지 않으면 default에 리소스가 생성됩니다.
- **kube-system**: Kubernetes 시스템 구성 요소들이 위치하는 네임스페이스.
- **kube-public**: 모든 사용자가 읽을 수 있는 리소스가 저장되는 네임스페이스.
- **kube-node-lease**: 노드에 대한 heartbeat 정보를 저장하는 네임스페이스.

Namespace를 활용하면 개발, 운영, 테스트 등의 환경을 분리하여 관리할 수 있습니다.

### Namespace 생성

새로운 네임스페이스는 `kubectl` 명령어를 사용하여 간단히 생성할 수 있습니다.

```bash
kubectl create namespace dev-ns
```

생성된 네임스페이스는 다음 명령어로 확인할 수 있습니다.

```bash
kubectl get namespaces
```

### Namespace 내 리소스 확인

특정 네임스페이스 내에 있는 리소스를 확인하려면 다음 명령을 사용할 수 있습니다.

```bash
kubectl get all -n dev-ns
```

## 2. 네임스페이스와 DNS

Kubernetes의 네임스페이스는 DNS와 통합되어 리소스 간 통신을 간편하게 해줍니다. 각 네임스페이스는 고유한 DNS 도메인을 가지며, 이를 통해 네임스페이스 간에도 명확한 통신을 할 수 있습니다. 예를 들어, 동일한 네임스페이스 내에서 Pod에 접근할 때는 서비스 이름만 사용하면 되지만, 다른 네임스페이스의 서비스에 접근하려면 다음과 같은 형식을 사용합니다:

```php
<service-name>.<namespace-name>.svc.cluster.local
```

### DNS 예시

다음은 다른 네임스페이스 간에 서비스에 접근하는 방법을 보여주는 예시입니다.

```bash
mysql.connect("mydb-svc.dev.svc.cluster.local")
```

이 예시에서는 `dev` 네임스페이스에 있는 `mydb-svc` 서비스를 참조하고 있습니다.

## 3. ResourceQuota 설정

**ResourceQuota**는 네임스페이스 내에서 사용할 수 있는 리소스의 최대치를 제한하는 방법입니다. ResourceQuota는 CPU, 메모리, 스토리지, 그리고 Pod 개수 등 다양한 자원에 대해 제한을 설정할 수 있습니다. 이를 통해 네임스페이스별로 자원 사용량을 조정하고 관리할 수 있습니다.

### ResourceQuota 예시

```yaml
yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: dev-ns
spec:
  hard:
    requests.cpu: "1000m"  # 총 1 CPU
    requests.memory: "1Gi" # 총 1GB 메모리
    limits.cpu: "2000m"    # 최대 2 CPU
    limits.memory: "2Gi"   # 최대 2GB 메모리
```

이 설정은 `dev-ns` 네임스페이스에서 CPU 사용량을 최대 2개 코어로 제한하고, 메모리 사용량은 최대 2GB로 제한하는 예시입니다.

### ResourceQuota 설정 및 적용

```bash
kubectl apply -f rq-1.yaml
```

적용된 ResourceQuota를 확인하려면 다음 명령을 사용합니다.

```bash
kubectl describe resourcequotas -n dev-ns
```

## 4. LimitRange 설정

**LimitRange**는 네임스페이스 내에서 Pod 또는 컨테이너가 사용할 수 있는 최소 및 최대 자원 요청 값을 설정할 수 있습니다. 이 기능은 리소스 사용량을 미리 정의하여 각 컨테이너가 적절한 자원을 할당받도록 보장합니다.

### LimitRange 예시

```yaml
yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limitr-1
  namespace: dev-ns
spec:
  limits:
    - type: Container
      max:
        cpu: "1"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "256Mi"
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
```

이 예시는 `dev-ns` 네임스페이스 내의 컨테이너가 최소 100m CPU, 256Mi 메모리를 사용하고, 최대 1 CPU와 1Gi 메모리를 사용할 수 있도록 설정합니다.

### LimitRange 설정 및 적용

```bash
kubectl apply -f limitr-1.yaml
```

적용된 LimitRange를 확인하려면 다음 명령을 사용합니다.

```bash
kubectl describe limitranges -n dev-ns
```

## 5. 네임스페이스 전환 및 기본 네임스페이스 설정

기본 네임스페이스를 변경하거나 특정 네임스페이스로 작업하려면 `kubectl config set-context` 명령을 사용하여 쉽게 전환할 수 있습니다.

### 네임스페이스 전환

```bash
kubectl config set-context --current --namespace=dev-ns
```

현재 설정된 네임스페이스를 확인하려면 다음 명령어를 실행합니다.

```bash
kubectl config view --minify | grep namespace:
```

이 명령을 통해 현재 작업 중인 네임스페이스가 무엇인지 확인할 수 있습니다.

## 6. 네임스페이스 활용 실습

Namespace를 활용한 실습을 통해 다양한 네임스페이스 간 리소스 관리 및 통신 설정 방법을 알아보겠습니다.

### 네임스페이스 간 서비스 통신

```bash
# ops-ns 네임스페이스에 nstest 배포
kubectl create deployment nstest -n ops-ns --image=nginx --port=80 --replicas=2
kubectl expose deployment nstest --name=nstest-svc --port=80 --target-port=80 -n ops-ns

# dev-ns 네임스페이스에 동일한 배포 생성
kubectl create deployment nstest -n dev-ns --image=nginx --port=80 --replicas=2
kubectl expose deployment nstest --name=nstest-svc --port=80 --target-port=80 -n dev-ns

# dev-ns와 ops-ns 간 통신 확인
curl http://nstest-svc.ops-ns.svc.cluster.local
curl http://nstest-svc.dev-ns.svc.cluster.local
```

이 실습에서는 두 개의 네임스페이스에 각각 서비스를 배포하고, 서로 다른 네임스페이스 간의 통신이 DNS를 통해 어떻게 이루어지는지 확인할 수 있습니다