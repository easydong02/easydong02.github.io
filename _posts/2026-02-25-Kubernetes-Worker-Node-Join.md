---
title: "[Kubernetes] 워커 노드 추가 완전 정리: 쌩 서버 vs AMI 복제"
date: 2026-02-25 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, kubeadm, worker-node, kubelet, containerd, calico, node, label, taint]
render_with_liquid: false
---

# Kubernetes 워커 노드 추가

## 1. 개요

**한 줄 요약**: 기존 Kubernetes 클러스터에 새로운 워커 노드를 추가하는 방법

**언제 사용하나**:
- 클러스터 리소스 확장이 필요할 때
- 특정 워크로드 전용 노드 추가 시
- 노드 장애 대응 또는 교체 시

### 방식 비교

| 항목 | 쌩 서버 (깡통) | AMI 복제 |
|------|---------------|----------|
| **소요 시간** | 30 ~ 60분 | 10 ~ 20분 |
| **난이도** | 중간 | 쉬움 |
| **장점** | 깨끗한 환경, 커스터마이징 | 빠름, 설정 일관성 |
| **단점** | 시간 소요, 실수 가능성 | 초기화 필수, machine-id 충돌 위험 |
| **사용 시기** | 최초 구축, 다른 OS 버전 | 운영 중 확장, 긴급 대응 |

> **실무 권장**: AMI 복제 방식이 훨씬 빠르고 안전합니다. 단, machine-id 재생성과 kubeadm reset이 핵심입니다.

**환경 정보**:
- Container Runtime: containerd 1.6.8
- Kubernetes: kubelet, kubeadm 1.21.14
- CNI: Calico 3.17.4
- OS: Amazon Linux 2

---

## 2. 방법 1: 쌩 서버 (깡통 서버)

### Step 1: OS 기본 설정

```bash
# 새 서버 기본 정보 확인
hostnamectl
ip addr show

# 1. hostname 설정
sudo hostnamectl set-hostname <new-node-name>

# 2. hosts 파일 수정
sudo vi /etc/hosts
# 추가:
# <NEW_NODE_IP>  <new-node-name>
# <MASTER_IP>    <master-hostname>

# 3. swap 비활성화 (Kubernetes 필수 요구사항)
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

# 4. SELinux 설정 (Amazon Linux 2)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

### Step 2: Container Runtime 설치 (containerd)

```bash
# 1. containerd 설치
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io-1.6.8

# 2. containerd 기본 설정 생성
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# 3. systemd cgroup 드라이버 설정 (Kubernetes 권장)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

# 4. containerd 시작
sudo systemctl enable containerd
sudo systemctl start containerd
sudo systemctl status containerd
```

### Step 3: Kubernetes 패키지 설치

```bash
# 1. Kubernetes repo 추가
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# 2. kubelet, kubeadm 설치 (버전 맞추기 중요!)
sudo yum install -y kubelet-1.21.14 kubeadm-1.21.14
sudo systemctl enable kubelet

# kubectl은 워커 노드에선 선택사항
# sudo yum install -y kubectl-1.21.14
```

### Step 4: 네트워크 설정 (Calico 준비)

```bash
# 1. br_netfilter 모듈 로드
sudo modprobe br_netfilter

# 2. 영구 설정
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

# 3. iptables 브리지 설정
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### Step 5: Master에서 Join Token 생성

```bash
# Master 노드에서 실행
kubeadm token create --print-join-command

# 출력 예시:
# kubeadm join 10.0.1.10:6443 --token abcdef.0123456789abcdef \
#   --discovery-token-ca-cert-hash sha256:1234567890abcdef...
```

**Token 만료 시 새로 생성**:

```bash
# Token 목록 확인 (24시간 유효)
kubeadm token list

# 새 Token 생성
kubeadm token create

# CA cert hash 확인 (재사용 가능)
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | \
  openssl rsa -pubin -outform der 2>/dev/null | \
  openssl dgst -sha256 -hex | sed 's/^.* //'
```

### Step 6: Worker 노드 Join

```bash
# 새 Worker 노드에서 실행
sudo kubeadm join <MASTER_IP>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>

# 성공 시 출력:
# [preflight] Running pre-flight checks
# [kubelet-start] Writing kubelet configuration to file
# [kubelet-start] Starting the kubelet
# This node has joined the cluster
```

### Step 7: Master에서 노드 확인

```bash
# 노드 추가 확인 (NotReady → Ready까지 1~3분 소요)
kubectl get nodes

# 출력 예시:
# NAME            STATUS     ROLES                  AGE    VERSION
# master-node     Ready      control-plane,master   100d   v1.21.14
# worker-node-1   Ready      <none>                 50d    v1.21.14
# new-worker      NotReady   <none>                 1m     v1.21.14  ← 신규

# Calico Pod 생성 확인
kubectl get pods -n kube-system -o wide | grep calico | grep <new-node-ip>

# 노드 상세 확인
kubectl describe node new-worker
```

### Step 8: Label 및 Taint 설정

```bash
# 1. 노드 역할 Label 추가
kubectl label node new-worker node-role.kubernetes.io/worker=worker

# 2. 특정 용도 Label 추가
kubectl label node new-worker workload=general

# 3. Taint 추가 (특정 Pod만 배치되도록 제한)
kubectl taint nodes new-worker node=aicwnd:NoSchedule

# 4. 확인
kubectl get nodes --show-labels
kubectl describe node new-worker | grep -i taint
```

---

## 3. 방법 2: AMI 복제한 서버

### 사전 체크

```bash
# AMI에 포함된 것 확인
sudo systemctl status containerd
sudo systemctl status kubelet
kubeadm version
kubelet --version
```

### Step 1: 노드별 고유 정보 초기화

```bash
# 1. hostname 변경
sudo hostnamectl set-hostname <new-node-name>

# 2. machine-id 재생성 (중요! 중복 시 노드 충돌 발생)
sudo rm -f /etc/machine-id /var/lib/dbus/machine-id
sudo systemd-machine-id-setup
cat /etc/machine-id  # 새 값 확인

# 3. kubelet 설정 삭제
sudo rm -rf /etc/kubernetes/kubelet.conf
sudo rm -rf /etc/kubernetes/pki/ca.crt
sudo rm -rf /var/lib/kubelet/*

# 4. containerd 초기화 (선택)
sudo systemctl stop kubelet
sudo systemctl stop containerd
sudo rm -rf /var/lib/containerd/*
sudo systemctl start containerd
```

### Step 2: 네트워크 설정 초기화

```bash
# 1. IP 주소 확인 (AWS는 자동 할당)
ip addr show

# 2. hosts 파일 수정
sudo vi /etc/hosts
# 자신의 새 IP와 hostname으로 수정

# 3. Calico 설정 초기화
sudo rm -rf /var/lib/calico
sudo rm -rf /etc/cni/net.d/*
```

### Step 3: 기존 클러스터 정보 제거

```bash
# 1. kubeadm reset (핵심!)
sudo kubeadm reset -f

# 성공 시 출력:
# [reset] Deleting contents of stateful directories
# [reset] Stopping the kubelet service

# 2. iptables 규칙 초기화
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X

# 3. ipvs 정리 (사용 시)
sudo ipvsadm --clear
```

### Step 4: Master에서 Join Token 생성

```bash
# Master 노드에서 실행 (방법 1과 동일)
kubeadm token create --print-join-command
```

### Step 5: Join 실행

```bash
sudo kubeadm join <MASTER_IP>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

### Step 6: Master에서 확인 및 Label/Taint 복제

```bash
# 1. 노드 확인
kubectl get nodes -o wide

# 2. 기존 노드 Label 확인
kubectl get nodes --show-labels

# 3. 기존 노드와 동일하게 Label 설정
kubectl label node <new-node> workload=general
kubectl label node <new-node> node-role.kubernetes.io/worker=worker

# 4. 기존 노드 Taint 확인 후 동일하게 설정
kubectl describe node <old-node> | grep Taints
kubectl taint nodes <new-node> node=aicwnd:NoSchedule
```

---

## 4. 공통: 검증 및 문제 해결

### 최종 검증

```bash
# 1. 모든 노드 Ready 확인
kubectl get nodes

# 2. 특정 노드에 테스트 Pod 배포
kubectl run test-pod --image=nginx \
  --overrides='{"spec":{"nodeSelector":{"kubernetes.io/hostname":"<new-node-name>"}}}'

# 3. Pod 상태 및 배치 확인
kubectl get pod test-pod -o wide

# 4. Pod 접속 테스트
kubectl exec -it test-pod -- curl localhost

# 5. 테스트 Pod 정리
kubectl delete pod test-pod
```

### 문제 해결

**NotReady 상태 지속 시**:

```bash
# Worker 노드에서 kubelet 로그 확인
sudo journalctl -u kubelet -f

# 흔한 에러:
# - "cni config uninitialized" → Calico 미설치
# - "node not found"           → API Server 연결 실패

# containerd 상태 확인
sudo systemctl status containerd

# CNI 플러그인 파일 확인
ls -la /opt/cni/bin/
ls -la /etc/cni/net.d/
```

**Calico Pod 시작 안 됨**:

```bash
# Master에서 Calico DaemonSet 확인
kubectl get ds -n kube-system calico-node

# Calico Pod 로그 확인
kubectl logs -n kube-system <calico-pod> -c calico-node

# NetworkPolicy 확인
kubectl get networkpolicy -A
```

**Join 실패 시**:

```bash
# Master와 통신 확인 (6443 포트)
telnet <MASTER_IP> 6443

# kubeadm reset 후 재시도
sudo kubeadm reset -f
sudo kubeadm join ...
```

---

## 5. 주의사항

**swap 활성화 상태로 Join**:

```
❌ swap 활성화된 채로 Join
→ kubelet 시작 실패

✅ swapoff -a 및 /etc/fstab 수정 필수
```

**machine-id 중복 (AMI 복제 시)**:

```
❌ machine-id 재생성 안 함
→ 노드 충돌, 네트워크 문제 발생

✅ systemd-machine-id-setup 반드시 실행
```

**kubeadm reset 없이 재Join (AMI 복제 시)**:

```
❌ 기존 설정 유지한 채로 Join
→ Join 실패 또는 이상 동작

✅ kubeadm reset -f 후 Join
```

**버전 불일치**:

```
❌ kubelet 1.22.x 설치
→ 클러스터 1.21.14와 호환 안 됨

✅ 클러스터 버전과 정확히 일치하는 버전 설치
```

**Label/Taint 미설정**:

```
❌ Label 없이 노드 추가
→ Pod가 의도치 않은 노드에 배치됨

✅ 기존 노드와 동일하게 Label/Taint 설정
```

---

## 6. 실무 체크리스트

### 쌩 서버 (깡통) 체크리스트

- [ ] OS 기본 설정 (hostname, swap 비활성화, SELinux)
- [ ] containerd 1.6.8 설치 및 systemd cgroup 설정
- [ ] kubelet, kubeadm 1.21.14 설치
- [ ] 네트워크 설정 (br_netfilter, iptables)
- [ ] Master에서 join token 생성
- [ ] `kubeadm join` 실행
- [ ] Label/Taint 설정
- [ ] 노드 상태 확인 (Ready)

### AMI 복제 체크리스트

- [ ] hostname 변경
- [ ] **machine-id 재생성** (핵심!)
- [ ] kubelet 설정 삭제
- [ ] Calico 설정 초기화
- [ ] **`kubeadm reset -f`** (핵심!)
- [ ] iptables 초기화
- [ ] Master에서 join token 생성
- [ ] `kubeadm join` 실행
- [ ] Label/Taint 기존 노드와 동일하게 복제
- [ ] 노드 상태 확인 (Ready)

---

## 7. 작업 전 백업

```bash
# Master 노드에서

# 1. 현재 노드 목록 백업
kubectl get nodes -o yaml > nodes-backup-$(date +%Y%m%d).yaml

# 2. etcd 스냅샷 백업 (안전한 방법)
ETCDCTL_API=3 etcdctl snapshot save snapshot-$(date +%Y%m%d).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

## 8. 운영계 적용 시 주의사항

운영계 워커 노드 추가는 반드시 아래 순서로 진행합니다.

1. **개발계에서 먼저 테스트** 완료 후 진행
2. **CSR 작성 및 승인** 대기 (작업 시간: 화/목 18:00 이후)
3. **작업 전 etcd 백업** 필수
4. **단계별 확인 및 기록** 작성

롤백 방법: 문제 발생 시 `kubectl delete node <new-node>`로 즉시 제거 가능

---

## 참고 자료

- 공식 문서: <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>