---
title: "[Kubernetes] 클러스터 환경 구축"
date: 2024-09-05 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, container, cluster, config]
render_with_liquid: false
---

## 🛠️ Kubernetes Cluster 환경 구축: 직접 해보는 3-node 클러스터

지난 포스팅에서 컨테이너 오케스트레이션의 강자, 쿠버네티스의 개념과 필요성에 대해 알아보았습니다. 이번에는 3개의 노드로 구성된 쿠버네티스 클러스터를 직접 구축해보면서 쿠버네티스의 세계에 첫 발을 내딛어 보겠습니다.

### 1. VM 환경 구성 (Ubuntu 22.04 기준)

먼저, 쿠버네티스 클러스터를 구성할 가상 머신(VM) 환경을 준비합니다.

- **Oracle VirtualBox 설치:** VirtualBox를 설치하고, '새로 만들기'를 선택하여 VM 생성을 시작합니다.
- **VM 사양 설정:** 쿠버네티스의 최소 요구 사양은 CPU 4개, 메모리 4GB입니다. 디스크 용량은 100GB로 설정하고, 네트워크 설정은 어댑터1(NAT), 어댑터2(Host-Only)로 구성합니다.
- **Ubuntu 설치:** 다운로드한 Ubuntu 22.04 ISO 파일을 VM에 연결하고, Ubuntu를 설치합니다.
- **네트워크 설정:** NAT와 Host-Only 네트워크 어댑터를 설정하고, 필요한 도구(vim, openssh-server, net-tools)를 설치합니다.
- **원격 접속 설정:** Putty, MobaXterm 등을 사용하여 VM에 원격 접속합니다.

### 2. Kubernetes를 위한 VM 환경 구성

이제 쿠버네티스를 설치하고 실행하기 위한 추가 설정을 진행합니다.

- **방화벽 해제 및 SWAP 비활성화**: `ufw disable` 명령어로 방화벽을 해제하고, `swapoff -a` 및 `/etc/fstab` 파일 수정을 통해 SWAP을 비활성화합니다.
- **NTP 설정**: `ntp` 패키지를 설치하고 NTP 서비스를 재시작하여 시간 동기화를 설정합니다.
- **IP 포워딩 활성화**: `/proc/sys/net/ipv4/ip_forward` 파일에 '1'을 입력하여 IP 포워딩을 활성화합니다.
- **컨테이너 런타임 구성**: `containerd`를 사용하여 컨테이너 런타임을 구성합니다.
- **iptables 설정**: 노드 간 통신을 위해 iptables에 브릿지 관련 설정을 추가합니다.
- **Docker 및 Kubernetes 도구 설치**: `docker-ce`, `containerd.io`, `kubelet`, `kubeadm`, `kubectl`을 설치합니다.
- **Kubernetes 도구 버전 확인 및 고정**: 설치된 Kubernetes 도구의 버전을 확인하고, 자동 업데이트를 방지하기 위해 `apt-mark hold` 명령어를 사용합니다.
- **kubelet 서비스 활성화**: `kubelet` 서비스를 활성화하고 재시작합니다.
- **hosts 파일 설정**: `/etc/hosts` 파일에 마스터 노드와 워커 노드의 IP 주소 및 호스트 이름을 추가합니다.
- **마스터 노드 복제**: VirtualBox UI를 통해 마스터 노드를 복제하여 워커 노드(k8s-node1, k8s-node2, k8s-node3)를 생성합니다.
- **워커 노드 설정**: 각 워커 노드의 호스트 이름과 IP 주소를 변경하고, `containerd` 및 `kubelet` 서비스를 재시작합니다.
- **SSH 연결 테스트**: 모든 노드 간의 SSH 연결을 테스트하여 통신이 원활하게 이루어지는지 확인합니다.

### 3. 3-node 클러스터를 위한 Kubernetes 초기화

이제 마스터 노드에서 쿠버네티스 클러스터를 초기화하고, 워커 노드를 클러스터에 추가합니다.

- **쿠버네티스 초기화**: `kubeadm init` 명령어를 사용하여 쿠버네티스 컨트롤 플레인을 초기화합니다. `pod-network-cidr` 및 `apiserver-advertise-address` 옵션을 적절히 설정합니다.
- **kubectl 설정**: `admin.conf` 파일을 복사하고 권한을 설정하여 `kubectl` 명령어를 사용할 수 있도록 합니다.
- **kubectl 자동 완성 기능 설치**: `bash-completion` 패키지를 설치하고, `kubectl completion bash` 명령어를 실행하여 자동 완성 기능을 활성화합니다.
- **네트워크 플러그인 설치**: `calico.yaml` 파일을 다운로드하고 `kubectl apply` 명령어를 사용하여 Calico 네트워크 플러그인을 설치합니다.
- **노드 상태 확인**: `kubectl get nodes` 명령어를 사용하여 모든 노드가 `Ready` 상태인지 확인합니다.
- **워커 노드 추가**: `kubeadm join` 명령어와 마스터 노드에서 제공하는 토큰 정보를 사용하여 워커 노드를 클러스터에 추가합니다.

### 4. Amazon EKS 환경 구성

Amazon EKS(Elastic Kubernetes Service)는 AWS에서 관리형 쿠버네티스 서비스를 제공하여 클러스터 구축 및 관리를 간소화합니다.

- **kubectl 설치**: `curl` 명령어를 사용하여 `kubectl`을 다운로드하고 설치합니다.
- **eksctl 설치**: `curl` 명령어를 사용하여 `eksctl`을 다운로드하고 설치합니다. `eksctl`은 EKS 클러스터 생성 및 관리를 위한 CLI 도구입니다.
- **AWS CLI 설정**: `aws configure` 명령어를 사용하여 AWS 액세스 키 ID, 비밀 액세스 키, 리전 등을 설정합니다.
- **EKS 클러스터 생성**: `eksctl create cluster` 명령어를 사용하여 EKS 클러스터를 생성합니다. 클러스터 이름, 노드 그룹 이름, 인스턴스 유형, 노드 수, 쿠버네티스 버전 등을 지정합니다.
