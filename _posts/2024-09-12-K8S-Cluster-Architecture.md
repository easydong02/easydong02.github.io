---
title: "[Kubernetes] 쿠버네티스 아키텍처"
date: 2024-09-12 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, calico, cni, architecture, containerd]
render_with_liquid: false
---

# Kubernetes 아키텍처 이해

Kubernetes는 애플리케이션을 컨테이너화하고 이를 관리할 수 있도록 설계된 오케스트레이션 플랫폼입니다. 이 플랫폼은 클러스터 환경에서 컨테이너화된 애플리케이션의 배포, 확장, 모니터링, 유지 관리를 자동화합니다. 이번 포스팅에서는 Kubernetes 아키텍처와 그 구성 요소들에 대해 자세히 알아보겠습니다.

## 1. Kubernetes 구조 이해

Kubernetes 아키텍처는 크게 두 가지 주요 구성 요소로 나눌 수 있습니다:

- **Control Plane**: 클러스터 전반을 관리하고 애플리케이션을 배포하며 상태를 유지하는 역할을 합니다.
- **Worker Nodes**: 실제 애플리케이션이 배포되고 실행되는 물리적/가상 서버들입니다.

각각의 구성 요소는 서로 협력하여 애플리케이션이 안정적으로 실행되고 적절한 리소스를 할당받도록 보장합니다.

### 1.1 Control Plane 구성 요소

1. **Kube-apiserver**: Kubernetes의 핵심 API 서버로, 모든 구성 요소들이 상호작용할 때 거치는 중심 역할을 합니다. 클라이언트(예: `kubectl`)에서 요청을 받으면, 이 요청을 처리하고 필요한 작업을 실행합니다.
2. **etcd**: 분산형 키-값 저장소로, 클러스터 상태 데이터를 저장하고 관리합니다. 모든 클러스터 관련 정보는 etcd에 저장되며, 이는 고가용성 환경을 구성하는 데 매우 중요한 역할을 합니다.
3. **Kube-controller-manager**: 클러스터의 상태를 유지하고 관리하는 컨트롤러들의 집합입니다. 예를 들어, 레플리카 컨트롤러는 항상 지정된 수의 파드를 유지하는 역할을 합니다.
4. **Kube-scheduler**: 파드를 적절한 노드에 할당하는 역할을 합니다. 스케줄러는 리소스 사용량과 가용성을 고려하여 적절한 노드를 선택합니다.
5. **Cloud-controller-manager**: Kubernetes 클러스터를 클라우드 환경에서 실행할 때, 클라우드 리소스와의 상호작용을 처리합니다.

### 1.2 Worker Node 구성 요소

1. **Kubelet**: 각 워커 노드에서 실행되며, 컨테이너가 원활하게 실행되도록 관리합니다. Kubelet은 apiserver로부터 받은 명령에 따라 컨테이너의 상태를 모니터링하고, 지정된 대로 실행됩니다.
2. **Kube-proxy**: Kubernetes 서비스와 파드 간의 네트워크 통신을 처리합니다. 각 노드에서 실행되며, 파드와 외부 세계 간의 통신이 가능하도록 지원합니다.
3. **Container Runtime**: Docker나 containerd와 같은 컨테이너 런타임은 실제로 컨테이너를 실행하는 역할을 합니다.

## 2. 애플리케이션 배포 과정

일반적인 애플리케이션의 Kubernetes 배포 과정은 다음과 같습니다:

1. **개발**: 개발자가 애플리케이션 코드를 작성하고 컨테이너 이미지로 빌드합니다. Dockerfile을 작성하여 필요한 애플리케이션 환경을 정의하고, `docker build` 명령어를 통해 이미지를 생성합니다.

```bash
# Dockerfile 예시
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]
```

1. **이미지 레지스트리 업로드**: 생성된 이미지를 퍼블릭 또는 프라이빗 Docker 레지스트리에 업로드합니다.

```bash
# Docker 이미지를 빌드하고 레지스트리에 푸시하는 명령
docker build -t myapp:v1 .
docker push myrepo/myapp:v1
```

1. **Kubernetes 배포**: Kubernetes 배포 파일(Pod, Deployment, Service)을 작성하여 클러스터에 배포합니다.

```yaml
# Deployment YAML 예시
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: myrepo/myapp:v1
        ports:
        - containerPort: 3000
```

1. **서비스 생성**: 애플리케이션이 외부와 통신할 수 있도록 Service 객체를 생성합니다. 클러스터 내부 통신을 위한 ClusterIP 또는 외부 접속을 위한 LoadBalancer 서비스 유형을 사용할 수 있습니다.

```yaml
# Service YAML 예시
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

1. **배포 확인 및 모니터링**: `kubectl` 명령어를 사용하여 배포된 애플리케이션을 확인하고, 상태를 모니터링합니다.

```bash
# 파드 상태 확인
kubectl get pods

# 서비스 상태 확인
kubectl get svc

# 특정 파드의 로그 확인
kubectl logs <pod-name>
```

## 3. 실습: CNI와 네트워크 설정 (Calico)

Kubernetes 클러스터에서 네트워크 통신을 관리하기 위해 CNI(Container Network Interface)를 설정해야 합니다. Calico는 Kubernetes에서 자주 사용되는 네트워크 플러그인 중 하나로, 네트워크 정책을 설정하고 네트워크 트래픽을 관리하는 역할을 합니다.

### Calico 설치

다음 명령을 통해 Kubernetes 클러스터에 Calico를 설치할 수 있습니다.

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

설치가 완료되면 Calico가 네트워크 트래픽을 관리하며, 클러스터 간 통신을 처리합니다.

### Calico 설정 확인

Calico 설정이 제대로 동작하는지 확인하기 위해 다음 명령을 사용할 수 있습니다.

```bash
# Calico 파드 상태 확인
kubectl get pods -n kube-system | grep calico

# 네트워크 라우팅 확인
kubectl exec -n kube-system calico-node -- calicoctl ipam show
```

## 4. 컨테이너 런타임(Container Runtime)

Kubernetes는 다양한 컨테이너 런타임을 지원하며, 대표적인 런타임은 Docker와 containerd입니다. 컨테이너 런타임은 컨테이너를 실행하고 관리하는 역할을 합니다. Kubernetes는 **CRI (Container Runtime Interface)**를 통해 런타임과 상호작용하며, gRPC 프로토콜을 사용하여 컨테이너를 시작, 중지 및 관리합니다.

### 런타임 동작 확인

Kubernetes에서 실행 중인 컨테이너 런타임을 확인하려면 다음 명령을 사용할 수 있습니다.

```bash
# containerd 동작 확인
pstree | grep containerd

# 파드가 containerd에서 어떻게 관리되고 있는지 확인
kubectl get pods -o wide
```