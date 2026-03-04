---
title: "[Kubernetes] Istio Service Mesh 완전 정리: 트래픽 관리, 보안, 가시성"
date: 2026-03-04 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, istio, service-mesh, envoy, sidecar, mtls, virtualservice, destinationrule, canary, circuit-breaker, kiali, jaeger]
render_with_liquid: false
---

# Istio Service Mesh

## 1. 개요

**한 줄 요약**: Kubernetes 클러스터에서 마이크로서비스 간 통신을 관리, 보안, 모니터링하는 Service Mesh 플랫폼

**핵심 가치**:
- 애플리케이션 코드 수정 없이 트래픽 관리, 보안, 가시성 제공
- Sidecar Proxy (Envoy) 패턴으로 모든 네트워크 트래픽 제어
- 마이크로서비스 아키텍처의 복잡성 해결

**언제 사용하나**:
- 마이크로서비스 간 통신 제어 (A/B 테스트, Canary 배포)
- 서비스 간 mTLS 암호화 및 인증
- 트래픽 라우팅, 재시도, 타임아웃 설정
- 분산 추적 (Distributed Tracing) 및 메트릭 수집

---

## 2. 아키텍처

### 2.1. 전체 구조

```
┌─────────────────────────────────────────────────────────────┐
│                     Istio Control Plane                     │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │  Pilot   │  │  Citadel │  │  Galley  │  (Istio 1.5+는     │
│  │(istiod)  │  │(istiod)  │  │(istiod)  │   통합: istiod)    │
│  └──────────┘  └──────────┘  └──────────┘                   │
│       │              │              │                        │
└───────┼──────────────┼──────────────┼────────────────────────┘
        │ 설정 전달      │ 인증서        │ 정책
        ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────┐
│                        Data Plane                           │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │    Pod A     │  │    Pod B     │  │    Pod C     │       │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │       │
│  │ │   App    │ │  │ │   App    │ │  │ │   App    │ │       │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │       │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │       │
│  │ │  Envoy   │ │  │ │  Envoy   │ │  │ │  Envoy   │ │       │
│  │ │(Sidecar) │ │  │ │(Sidecar) │ │  │ │(Sidecar) │ │       │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
│          │                 │                 │              │
│          └─────────────────┼─────────────────┘              │
│                   모든 트래픽 Envoy 경유                        │
└─────────────────────────────────────────────────────────────┘
```

### 2.2. Control Plane (istiod)

| 컴포넌트 | 역할 | 기능 |
|---------|------|------|
| **Pilot** | 서비스 디스커버리 & 트래픽 관리 | Envoy 설정 배포, 라우팅 규칙 |
| **Citadel** | 인증서 관리 | mTLS 인증서 발급/갱신 |
| **Galley** | 설정 검증 & 배포 | Istio 리소스 검증 |

Istio 1.5+부터 세 컴포넌트가 `istiod` 단일 바이너리로 통합되었습니다.

### 2.3. Data Plane (Envoy Sidecar)

Sidecar Injection 후 Pod에 `istio-proxy` 컨테이너가 자동으로 추가됩니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp           # 애플리케이션 컨테이너
    image: myapp:1.0
  - name: istio-proxy     # Envoy Sidecar (자동 주입)
    image: docker.io/istio/proxyv2:1.5.10
```

**Envoy Proxy 역할**:
- 모든 인바운드/아웃바운드 트래픽 가로채기
- 라우팅, 로드밸런싱, 재시도, 타임아웃 적용
- mTLS 암호화/복호화
- 메트릭, 로그, 분산 추적 데이터 생성

---

## 3. 주요 기능

### 3.1. 트래픽 관리

#### VirtualService: 라우팅 규칙 정의

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews.default.svc.cluster.local
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome.*"
    route:
    - destination:
        host: reviews
        subset: v2      # Chrome 사용자 → v2
      weight: 100
  - route:
    - destination:
        host: reviews
        subset: v1      # 나머지 → v1
      weight: 100
```

#### DestinationRule: 서브셋 및 트래픽 정책

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews.default.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN      # 로드밸런싱 알고리즘
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 2
    outlierDetection:          # Circuit Breaker
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**주요 트래픽 관리 기능**:
- **A/B 테스트**: 헤더 기반 라우팅
- **Canary 배포**: 트래픽 가중치 조정 (v1: 90%, v2: 10%)
- **Circuit Breaker**: 장애 서비스 격리
- **재시도 & 타임아웃**: 네트워크 장애 대응
- **요청 미러링**: 프로덕션 트래픽을 테스트 환경으로 복사

### 3.2. 보안 (mTLS)

#### PeerAuthentication: 서비스 간 mTLS 강제

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT      # STRICT: mTLS 필수
                      # PERMISSIVE: mTLS/평문 모두 허용
                      # DISABLE: mTLS 비활성화
```

**mTLS 동작 방식**:

```
Pod A (Envoy) → mTLS 암호화 → Pod B (Envoy)
    ↑                              ↓
Citadel 인증서 발급         Citadel 인증서 검증
```

#### AuthorizationPolicy: 접근 제어

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: reviews-viewer
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/reviews/*"]
```

**보안 기능 요약**:
- **자동 mTLS**: 인증서 자동 발급/갱신 (90일)
- **서비스 간 인증**: SPIFFE ID 기반
- **세밀한 접근 제어**: 네임스페이스, ServiceAccount, HTTP 메서드 기반

### 3.3. 가시성 (Observability)

#### 메트릭 자동 수집 (Prometheus)

```promql
# 요청 수
istio_requests_total{destination_service="reviews"}

# 요청 지연 시간
istio_request_duration_milliseconds{destination_service="reviews"}

# 요청/응답 크기
istio_request_bytes{destination_service="reviews"}
istio_response_bytes{destination_service="reviews"}
```

#### Kiali: 서비스 메시 시각화

```bash
istioctl dashboard kiali
# 기능:
# - 서비스 토폴로지 그래프
# - 트래픽 흐름 시각화
# - VirtualService, DestinationRule 설정 확인
# - 메트릭 및 추적 데이터 연동
```

#### Jaeger: 분산 추적

```
frontend → productpage → reviews → ratings
   │           │            │          │
   └───────────┴────────────┴──────────┘
            Trace ID: abc123

Jaeger UI:
- 전체 요청 경로 시각화
- 각 서비스별 지연 시간
- 에러 발생 지점 파악
```

---

## 4. 설치 및 기본 설정

### 4.1. Istio 설치

```bash
# Istio 다운로드
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.5.10
export PATH=$PWD/bin:$PATH

# 설치 프로파일 목록 확인
istioctl profile list
# - default : 프로덕션 권장
# - demo    : 테스트/학습용 (모든 기능 활성화)
# - minimal : 최소 구성
# - remote  : Multi-cluster

# 설치 (demo 프로파일)
istioctl install --set profile=demo -y

# 설치 확인 (istiod, ingressgateway, egressgateway)
kubectl get pods -n istio-system
```

### 4.2. Sidecar 자동 주입 활성화

```bash
# 네임스페이스에 Label 추가
kubectl label namespace default istio-injection=enabled

# 확인
kubectl get namespace -L istio-injection

# 기존 Pod에 적용 (재배포)
kubectl rollout restart deployment <deployment-name>

# 특정 Deployment만 수동 주입
kubectl get deployment productpage -o yaml \
  | istioctl kube-inject -f - \
  | kubectl apply -f -
```

---

## 5. 기본 명령어

```bash
# Istio 버전 확인
istioctl version

# Istio 설정 검증 (전체 네임스페이스)
istioctl analyze -n default

# Proxy 상태 전체 확인
istioctl proxy-status

# Pod별 Proxy 설정 확인
istioctl proxy-config routes    <pod-name> -n <namespace>
istioctl proxy-config clusters  <pod-name> -n <namespace>
istioctl proxy-config listeners <pod-name> -n <namespace>

# VirtualService 검증
istioctl analyze virtualservice/reviews -n default

# 대시보드 실행
istioctl dashboard kiali
istioctl dashboard jaeger
istioctl dashboard grafana
istioctl dashboard prometheus

# Envoy 로그 레벨 변경 (디버깅 시)
istioctl proxy-config log <pod-name> --level debug
```

---

## 6. 실무 패턴

### 6.1. Canary 배포

```yaml
# VirtualService: 트래픽 가중치 조정
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-canary
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90    # 기존 버전 90%
    - destination:
        host: reviews
        subset: v2
      weight: 10    # 신규 버전 10%
---
# DestinationRule: 서브셋 정의
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

**배포 프로세스**: v2 배포 (10%) → 모니터링 (에러율, 지연 시간) → 단계적 증가 (10% → 30% → 50% → 100%) → v1 제거

### 6.2. 타임아웃 & 재시도

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews-retry
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
    timeout: 3s           # 3초 타임아웃
    retries:
      attempts: 3         # 최대 3회 재시도
      perTryTimeout: 1s   # 재시도마다 1초 대기
      retryOn: 5xx        # 5xx 에러 시에만 재시도
```

### 6.3. Circuit Breaker

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews-circuit-breaker
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        maxRequestsPerConnection: 2
    outlierDetection:
      consecutiveErrors: 5      # 5회 연속 실패 시
      interval: 30s             # 30초마다 평가
      baseEjectionTime: 30s     # 30초간 격리
      maxEjectionPercent: 50    # 최대 50% Pod만 격리
```

### 6.4. Ingress Gateway

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.example.com"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "bookinfo.example.com"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        prefix: "/productpage"
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

```
외부 사용자 → Istio Ingress Gateway → VirtualService 라우팅 → productpage Service
```

### 6.5. 네임스페이스 격리 (mTLS)

```yaml
# production: STRICT (외부 평문 트래픽 차단)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: production-mtls
  namespace: production
spec:
  mtls:
    mode: STRICT
---
# dev: PERMISSIVE (테스트 편의)
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: dev-mtls
  namespace: dev
spec:
  mtls:
    mode: PERMISSIVE
```

---

## 7. 금융권 환경 보안 강화 설정

```yaml
# 1. 전 클러스터 mTLS STRICT 적용
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: global-mtls
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
---
# 2. Egress 화이트리스트 (외부 접근 허용 대상만 등록)
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-api-whitelist
spec:
  hosts:
  - "api.trusted-partner.com"
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
---
# 3. AuthorizationPolicy: production 네임스페이스 외 접근 거부
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: production
spec:
  action: DENY
  rules:
  - from:
    - source:
        notNamespaces: ["production"]
```

---

## 8. 모니터링

### 8.1. Prometheus 핵심 쿼리

```promql
# 요청 성공률
sum(rate(istio_requests_total{response_code!~"5.*"}[1m]))
/
sum(rate(istio_requests_total[1m]))

# P99 지연 시간 (서비스별)
histogram_quantile(0.99,
  sum(rate(istio_request_duration_milliseconds_bucket[1m]))
  by (le, destination_service)
)

# Circuit Breaker 발동 횟수
rate(istio_requests_total{response_flags=~".*UO.*"}[1m])
```

### 8.2. Grafana 주요 대시보드

```bash
istioctl dashboard grafana
# - Istio Mesh Dashboard    : 전체 메시 상태
# - Istio Service Dashboard : 서비스별 메트릭
# - Istio Workload Dashboard: Pod별 메트릭
# - Istio Performance Dashboard: Control Plane 성능
```

---

## 9. 트러블슈팅

### 9.1. Sidecar 주입 안 됨

```bash
# 원인 확인: 네임스페이스 Label 누락
kubectl get namespace default -o yaml | grep istio-injection
# istio-injection: enabled 없으면 주입 안 됨

# 해결
kubectl label namespace default istio-injection=enabled
kubectl rollout restart deployment <deployment-name>
```

### 9.2. VirtualService 적용 안 됨

```bash
# 설정 검증
istioctl analyze virtualservice/reviews -n default

# Envoy 라우팅 설정 확인
istioctl proxy-config routes <pod-name> -n default

# 흔한 원인:
# - host 불일치 (FQDN 확인: reviews.default.svc.cluster.local)
# - subset 라벨 불일치
# - Gateway 연결 누락
```

### 9.3. mTLS 연결 실패 (503)

```bash
# PeerAuthentication 확인
kubectl get peerauthentication -A

# DestinationRule에 TLS 모드 명시
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
EOF
```

### 9.4. 메트릭 수집 안 됨

```bash
# Envoy Stats 직접 확인
kubectl exec -it <pod-name> -c istio-proxy -- curl localhost:15000/stats/prometheus

# Telemetry 설정 확인 (Istio 1.5+)
kubectl get telemetry -A

# Prometheus 서비스 확인
kubectl get svc -n istio-system prometheus
```

### 9.5. 성능 저하 (Sidecar 리소스 과다)

```bash
# Sidecar 리소스 사용량 확인
kubectl top pod -n default --containers | grep istio-proxy

# Sidecar 리소스 제한 조정
istioctl install \
  --set values.global.proxy.resources.requests.cpu=200m \
  --set values.global.proxy.resources.requests.memory=256Mi
```

---

## 10. 운영 참고사항

### Istio 버전별 Kubernetes 지원 범위

| Istio 버전 | Kubernetes 지원 | 주요 변경 사항 |
|-----------|----------------|--------------|
| **1.5.x** | 1.14 - 1.18 | istiod 통합, Mixer 제거 |
| **1.10.x** | 1.18 - 1.22 | Gateway API 지원 |
| **1.15.x** | 1.21 - 1.25 | Ambient Mesh (Sidecar-less, 실험적) |

HyperCloud (K8s 1.21) 환경에서는 Istio **1.5.x ~ 1.10.x**를 사용하며, 현재 운영 환경은 **Istio 1.5.10**입니다.

### Sidecar 리소스 오버헤드

- CPU: Pod당 50~200m 추가
- 메모리: Pod당 128~512Mi 추가
- 지연 시간: 평균 1~3ms 증가
- 대규모 환경에서는 리소스 계획 필수

### Istio vs 대안 기술 비교

| 도구 | 특징 | 적합한 상황 |
|------|------|------------|
| **Istio** | 풍부한 기능, 높은 복잡도 | 대규모 마이크로서비스, 엄격한 보안 요구 |
| **Linkerd** | 경량, 단순, Rust 기반 | 낮은 오버헤드 우선 환경 |
| **Consul Connect** | HashiCorp 생태계 통합 | Vault, Consul 사용 중인 환경 |
| **AWS App Mesh** | AWS 관리형 | AWS 전용 환경 |

마이크로서비스가 10개 미만이라면 Istio 없이 Ingress + cert-manager 조합이 더 단순할 수 있습니다.

---

## 참고 자료

- Istio 공식 문서: <https://istio.io/latest/docs/>
- Envoy Proxy: <https://www.envoyproxy.io/>
- Kiali: <https://kiali.io/>
- Bookinfo 예제: <https://istio.io/latest/docs/examples/bookinfo/>