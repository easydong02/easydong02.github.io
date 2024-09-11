---
title: "[Kubernetes] 쿠버네티스 관리 도구"
date: 2024-09-11 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, container, cluster, config]
render_with_liquid: false
---

# Kubernetes 관리 도구: Kubernetes Dashboard, Prometheus, Grafana, Kubeshark, Portainer

Kubernetes는 클러스터 환경에서 애플리케이션을 효율적으로 관리할 수 있도록 다양한 관리 도구를 제공합니다. 이번 포스팅에서는 Kubernetes 관리 도구 중 Kubernetes Dashboard, Prometheus, Grafana, Kubeshark, Portainer에 대해 살펴보겠습니다. 이 도구들은 Kubernetes 클러스터를 모니터링하고, 리소스 상태를 확인하며, 클러스터 환경을 보다 쉽게 관리하는 데 큰 도움을 줍니다.

## 1. Kubernetes Dashboard

Kubernetes Dashboard는 Kubernetes 클러스터를 시각적으로 관리할 수 있는 웹 UI입니다. Kubernetes 리소스를 한눈에 확인하고, 직접 애플리케이션을 관리할 수 있어 매우 유용합니다. 이를 통해 클러스터에 배포된 애플리케이션, 컨테이너, 네트워크 설정, 서비스 등을 쉽게 관리할 수 있습니다.

### ☑️Kubernetes Dashboard 설치

Kubernetes Dashboard를 설치하려면 다음 명령을 실행합니다.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

설치가 완료되면 `kubectl get po -A` 명령을 사용하여 Dashboard와 관련된 포드를 확인할 수 있습니다.

```bash
kubectl get po -A
```

설치된 후에는 적합한 권한이 필요합니다. `cluster-admin` 역할을 부여하여 전역 관리 권한을 설정합니다.

```bash
kubectl get clusterrole cluster-admin
kubectl describe clusterrole cluster-admin
```

### ☑️Kubernetes Dashboard 접속

Dashboard에 접속하기 위해 생성된 토큰을 사용해야 합니다. `kubectl create token` 명령을 통해 토큰을 생성하고, 이를 사용하여 Dashboard에 로그인할 수 있습니다.

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Dashboard URL에 접속하여 로그인 후 Kubernetes 클러스터를 관리할 수 있습니다.

---

## 2. Prometheus & Grafana

Prometheus는 모니터링 시스템이며, Grafana는 데이터를 시각화하는 도구입니다. 이 두 도구는 Kubernetes 클러스터의 성능을 모니터링하고, 다양한 리소스의 상태를 확인하는 데 사용됩니다.

### ☑️Prometheus 설치

다음 명령을 사용하여 Prometheus를 설치합니다.

```bash
kubectl create namespace monitoring
kubectl create -f prometheus/prometheus-ConfigMap.yaml
kubectl create -f prometheus/prometheus-ClusterRole.yaml
kubectl create -f prometheus/prometheus-Deployment.yaml
kubectl create -f prometheus/prometheus-Service.yaml
```

### ☑️Grafana 설치

Grafana는 Prometheus 데이터를 시각화하는 데 사용됩니다. 다음 명령으로 Grafana를 설치할 수 있습니다.

```bash
kubectl create -f grafana/grafana-Deployment.yaml
kubectl create -f grafana/grafana-Service.yaml
```

설치가 완료되면 Grafana UI를 통해 시각화 대시보드를 설정할 수 있습니다. Grafana 대시보드에서 Prometheus를 데이터 소스로 연결한 후, 다양한 대시보드를 불러와 시각적으로 모니터링할 수 있습니다.

---

## 3. Kubeshark

Kubeshark는 Kubernetes 클러스터의 네트워크 트래픽을 모니터링하는 도구입니다. 클러스터 내에서 발생하는 API 호출이나 네트워크 패킷을 실시간으로 분석하고 시각화할 수 있습니다.

### ☑️Kubeshark 설치

Kubeshark를 설치하려면 다음 명령을 실행합니다.

```bash
sh <(curl -Ls https://kubeshark.co/install)
```

설치 후에는 `kubeshark tap` 명령을 사용하여 네트워크 트래픽을 모니터링할 수 있습니다.

```bash
kubeshark tap
```

이 명령은 클러스터 내 모든 트래픽을 분석하여 실시간으로 모니터링할 수 있는 환경을 제공합니다.

---

## 4. Portainer

Portainer는 Kubernetes 클러스터와 컨테이너를 관리하는 도구입니다. Kubernetes 리소스를 GUI를 통해 관리할 수 있으며, Kubernetes 뿐만 아니라 Docker 클러스터까지 관리할 수 있습니다.

### ☑️Portainer 설치

다음 명령으로 Portainer를 설치합니다.

```bash
kubectl apply -n portainer -f https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml
```

설치가 완료된 후에는 웹 UI를 통해 포트 9000번에 접속하여 Portainer를 사용할 수 있습니다. Portainer는 Kubernetes 리소스뿐만 아니라 스토리지, 네트워크, 볼륨 관리 등도 손쉽게 수행할 수 있는 기능을 제공합니다.