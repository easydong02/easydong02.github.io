---
title: "[Kubernetes] APIService 완전 정리: API Aggregation Layer와 확장 메커니즘"
date: 2026-02-25 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, apiservice, api-aggregation, metrics-server, crd, kube-apiserver, extension-api-server]
render_with_liquid: false
---

# Kubernetes APIService

## 1. 개요

Kubernetes에서 **APIService**는 쿠버네티스 API를 확장하기 위한 핵심 리소스입니다. 기본 `kube-apiserver`가 처리하지 않는 **별도의 API 그룹과 버전을 다른 서비스(Extension API Server)로 연결(Proxy)해주는 역할**을 합니다.

이를 이해하려면 먼저 **API Aggregation Layer(API 집계 계층)** 개념이 필요합니다.

---

## 2. APIService의 핵심 역할

쿠버네티스 클러스터에는 기본 리소스(Pod, Service, Node 등)를 처리하는 메인 `kube-apiserver`가 있습니다. 하지만 사용자가 만든 **커스텀 API**나 **Metrics Server** 같은 확장 기능을 메인 서버에 직접 통합하는 것은 어렵습니다.

**APIService** 리소스를 생성하면 다음과 같은 일이 일어납니다.

1. `kube-apiserver`에게 "특정 API 그룹(예: `metrics.k8s.io`)으로 들어오는 요청은 내가 처리하지 않고, **지정된 다른 서비스(Pod)로 프록시**하겠다"고 등록합니다.
2. 사용자가 `kubectl get --raw /apis/metrics.k8s.io/...`를 호출하면, 메인 API 서버는 이 요청을 실제 로직이 있는 **Extension API Server**로 전달합니다.

---

## 3. 주요 구성 요소와 동작 방식

```
┌────────────────────────────────────────────────────────────┐
│                   kube-apiserver                           │
│                                                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │             Aggregation Layer (집계 계층)             │  │
│  │  - 핵심 API 요청 → 직접 처리                          │  │
│  │  - 등록된 APIService 요청 → Extension Server로 프록시  │  │
│  └──────────────────────┬──────────────────────────────┘  │
└─────────────────────────┼──────────────────────────────────┘
                          │ 프록시
                          ▼
┌────────────────────────────────────────────────────────────┐
│              Extension API Server (Pod)                    │
│  - 실제 로직을 수행하는 별도의 Pod                            │
│  - 예: Metrics Server, Custom API Server                   │
└────────────────────────────────────────────────────────────┘
```

각 구성 요소의 역할은 다음과 같습니다.

- **Aggregation Layer (집계 계층)**: `kube-apiserver` 내부에 존재하며, 들어오는 요청이 핵심 API인지, 등록된 APIService인지 판단합니다.
- **Extension API Server**: 실제 로직을 수행하는 별도의 Pod입니다. (예: Metrics Server, Custom API Server)
- **APIService 리소스**: 집계 계층과 Extension Server를 연결해주는 '등록증' 역할을 하는 YAML 명세입니다.

---

## 4. 실전 예시: Metrics Server

가장 흔하게 접하는 APIService는 `kubectl top` 명령어를 가능하게 하는 **Metrics Server**입니다.

Metrics Server를 설치하면 클러스터에 다음과 같은 APIService가 생성됩니다.

```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  # 1. 이 서비스가 처리할 API 그룹과 버전 우선순위
  group: metrics.k8s.io
  version: v1beta1
  groupPriorityMinimum: 100
  versionPriority: 100

  # 2. 실제 요청을 처리할 쿠버네티스 Service 정보 (Extension API Server)
  service:
    name: metrics-server
    namespace: kube-system
    port: 443

  # 3. 보안 통신을 위한 CA 인증서 (메인 API 서버가 확장 서버를 신뢰하기 위함)
  caBundle: LS0tLS1...
  insecureSkipTLSVerify: false
```

**동작 흐름**: 사용자가 `metrics.k8s.io/v1beta1` API를 호출하면, `kube-apiserver`는 이 요청을 `kube-system` 네임스페이스의 `metrics-server` 서비스의 `443` 포트로 전달합니다.

---

## 5. APIService vs CRD (Custom Resource Definition)

쿠버네티스를 확장하는 두 가지 방법을 비교하면 아래와 같습니다.

| 특징 | CRD (Custom Resource Definition) | APIService (API Aggregation) |
|------|----------------------------------|------------------------------|
| **주 목적** | 새로운 데이터를 저장하고 조회할 때 | 데이터를 **실시간 연산**하거나, 별도 저장소를 쓸 때 |
| **데이터 저장** | etcd (쿠버네티스 기본 저장소) | **자유** (메모리, 외부 DB, 파일 등) |
| **난이도** | 쉬움 (YAML 정의만 하면 끝) | 어려움 (별도의 API 서버를 Go 등으로 개발해야 함) |
| **대표 예시** | Istio VirtualService, ArgoCD Application | Metrics Server (실시간 메모리 조회), Service Catalog |

**APIService를 써야 하는 경우**:
- 데이터를 etcd에 저장하지 않고 실시간으로 계산해서 보여줘야 할 때 (예: CPU 사용량)
- 기존 레거시 시스템을 쿠버네티스 API처럼 포장해서 사용하고 싶을 때
- Protobuf 같은 바이너리 프로토콜을 사용하여 성능을 극대화해야 할 때

---

## 6. 특정 명령어가 어떤 APIService를 사용하는지 파악하기

### 6.1. `-v=6` 옵션으로 API 경로 확인

`kubectl` 명령어 뒤에 `-v` 옵션(verbosity)을 붙이면, 이 명령어가 API 서버에 **어떤 URL로 요청을 보내는지** 로그를 볼 수 있습니다.

```bash
kubectl top nodes -v=6
```

출력 예시 (중요 부분만 발췌):

```
I0115 ... round_trippers.go:423] GET https://127.0.0.1:6443/apis/metrics.k8s.io/v1beta1/nodes
I0115 ... round_trippers.go:423] Request Headers:
```

**해석**: `GET` 뒤의 URL `/apis/metrics.k8s.io/v1beta1/nodes`를 보면 다음을 알 수 있습니다.
- API 그룹: `metrics.k8s.io`
- 버전: `v1beta1`
- APIService 이름: `v1beta1.metrics.k8s.io` (그룹 + 버전 조합)

### 6.2. APIService 리소스 확인

위에서 알아낸 이름으로 실제 APIService 리소스를 조회합니다.

```bash
# APIService 상태 확인
kubectl get apiservice v1beta1.metrics.k8s.io

# 연결된 실제 서비스(Pod) 확인
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
```

출력 결과 분석:

```yaml
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io   # URL에서 확인한 이름
spec:
  group: metrics.k8s.io
  version: v1beta1
  service:
    name: metrics-server          # 실제로 이 서비스로 프록시
    namespace: kube-system
    port: 443
```

### 6.3. 전체 추적 과정 요약

```
1. 명령어 실행
   kubectl top nodes

2. API 경로 확인 (-v=6)
   /apis/metrics.k8s.io/v1beta1/nodes

3. APIService 이름 매핑
   metrics.k8s.io + v1beta1 = v1beta1.metrics.k8s.io

4. 라우팅 확인
   APIService 설정 → kube-system/metrics-server:443 으로 트래픽 전달
```

> **팁**: `-v=6` 옵션은 `kubectl top` 뿐만 아니라 **모든 `kubectl` 명령어가 실제로 어떤 API를 호출하는지 궁금할 때** 사용할 수 있는 만능 디버깅 방법입니다.

---

## 7. 자주 쓰는 APIService 확인 명령어

```bash
# 전체 APIService 목록 조회
kubectl get apiservice

# 특정 APIService 상세 확인
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml

# APIService 상태 확인 (Available 여부)
kubectl get apiservice | grep -v Local

# 모든 API 그룹 확인
kubectl api-versions

# 특정 그룹의 리소스 확인
kubectl api-resources --api-group=metrics.k8s.io
```

출력 예시:

```
NAME                                   SERVICE                      AVAILABLE   AGE
v1.                                    Local                        True        100d
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        90d
v1beta1.custom.example.com             default/custom-api-server    True        30d
```

- `Local`: 메인 `kube-apiserver`가 직접 처리
- `kube-system/metrics-server`: 해당 Service로 프록시

---

## 8. 핵심 정리

**APIService**는 쿠버네티스 API 서버를 확장하여, **특정 URL 경로로 들어오는 요청을 별도로 구축된 API 서버(Pod)로 라우팅해주는 리소스**입니다.

| 구성 요소 | 역할 |
|-----------|------|
| **APIService 리소스** | API 그룹/버전과 Extension Server를 연결하는 등록증 |
| **Aggregation Layer** | 요청을 핵심 API로 처리할지, Extension Server로 프록시할지 판단 |
| **Extension API Server** | 실제 로직을 수행하는 별도의 Pod (예: metrics-server) |

**언제 APIService가 필요한가**: etcd가 아닌 방식의 데이터 처리가 필요하거나 (실시간 메트릭 등), 기존 레거시 시스템을 Kubernetes API처럼 노출시키고 싶을 때 사용합니다.

---

## 참고 자료

- Kubernetes API Aggregation 공식 문서: <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/>
- APIService API 레퍼런스: <https://kubernetes.io/docs/reference/kubernetes-api/cluster-resources/api-service-v1/>