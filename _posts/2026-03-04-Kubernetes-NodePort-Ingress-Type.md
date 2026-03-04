---
title: "[Kubernetes] NodePort vs Ingress 완전 정리: 외부 트래픽 노출 방식 비교와 실무 패턴"
date: 2026-03-04 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, nodeport, ingress, service, traefik, networking, tls, routing, load-balancer]
render_with_liquid: false
---

# NodePort vs Ingress

## 1. 개요

**한 줄 요약**: 외부에서 Kubernetes 클러스터 내부 서비스로 접근하기 위한 두 가지 방식

**언제 사용하나**:
- **NodePort**: 개발/테스트 환경, 특정 포트로 직접 접근, HTTP/HTTPS 외 프로토콜
- **Ingress**: 프로덕션 환경, 도메인 기반 HTTP/HTTPS 라우팅, SSL 종료

**관련 기술**: Service, Ingress Controller (Traefik, Nginx), LoadBalancer, DNS

---

## 2. 핵심 개념

### 2.1. 기본 접근 방식 비교

**NodePort**

```
외부 → 노드 IP:NodePort → Service → Pod
```

- 클러스터의 **모든 노드**에 동일한 포트 오픈 (30000~32767)
- 노드 IP를 알아야 접근 가능
- L4 레벨 (TCP/UDP)

**Ingress**

```
외부 → DNS/도메인 → Ingress Controller → Service → Pod
```

- HTTP/HTTPS 라우팅 (L7)
- 하나의 IP/도메인으로 여러 서비스 접근
- Path 기반, Host 기반 라우팅

### 2.2. 주요 차이점

| 항목 | NodePort | Ingress |
|------|----------|---------|
| **OSI 계층** | L4 (Transport) | L7 (Application) |
| **프로토콜** | TCP, UDP | HTTP, HTTPS |
| **포트 범위** | 30000~32767 | 80, 443 |
| **라우팅** | 불가능 | Path, Host 기반 라우팅 |
| **SSL/TLS** | 불가능 | 지원 (Ingress에서 종료) |
| **접근 방식** | `<NodeIP>:<NodePort>` | `http(s)://domain.com` |
| **사용 환경** | 개발/테스트 | 프로덕션 |
| **외부 의존성** | 없음 | Ingress Controller 필요 |

### 2.3. Ingress vs Ingress Controller 구분

Ingress 리소스 자체는 트래픽을 직접 받지 않습니다. Traefik이 읽고 적용하는 **라우팅 지시서(규칙 정의)** 역할만 합니다.

| 항목 | Ingress | Ingress Controller |
|------|---------|-------------------|
| **종류** | Kubernetes 리소스 (설정) | 실제 Pod (프로그램) |
| **역할** | "어디로 보낼지" 규칙 정의 | 규칙을 읽고 실제 라우팅 실행 |
| **예시** | YAML 매니페스트 | Traefik, Nginx 등 |
| **위치** | etcd에 저장 | Worker Node에서 실행 |

### 2.4. Ingress가 필요한 이유

**NodePort만 사용하면** 서비스마다 다른 포트가 필요합니다.

```
Service 1: NodePort 31001  (app1.example.com)
Service 2: NodePort 31002  (app2.example.com)
Service 3: NodePort 31003  (app3.example.com)
```

**Ingress를 사용하면** NodePort는 Traefik 하나만 열고, 도메인/Path 기반으로 자유롭게 라우팅할 수 있습니다.

```
Traefik NodePort: 31443 (하나만)

Ingress 규칙:
  app1.example.com       → app1-service
  app2.example.com       → app2-service
  app3.example.com/api   → app3-service
  app3.example.com/admin → admin-service
```

---

## 3. 기본 명령어

```bash
# NodePort Service 확인
kubectl get svc -n <namespace>
kubectl describe svc <service-name> -n <namespace>

# Ingress 확인
kubectl get ingress -n <namespace>
kubectl describe ingress <ingress-name> -n <namespace>

# Ingress Controller 확인 (HyperCloud는 Traefik)
kubectl get pods -n traefik-system

# NodePort 접근 테스트
curl http://<node-ip>:<node-port>

# Ingress 접근 테스트 (Host 헤더 지정)
curl -H "Host: app.example.com" http://<ingress-controller-ip>
curl http://app.example.com
```

**출력 예시**:

```
# NodePort Service
NAME         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
my-app-svc   NodePort   10.96.100.50    <none>        8080:31234/TCP   5d

# Ingress
NAME         CLASS    HOSTS             ADDRESS         PORTS     AGE
my-app-ing   traefik  app.example.com   192.168.1.100   80, 443   5d
```

---

## 4. 실무 패턴

### 4.1. NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
  namespace: default
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 8080        # Service 내부 포트
      targetPort: 8080  # Pod 포트
      nodePort: 31234   # 노드에서 열릴 포트 (생략 시 자동 할당)
```

```bash
# 클러스터 노드 IP 확인
kubectl get nodes -o wide

# 어느 노드로 접근해도 동일하게 동작
curl http://192.168.1.10:31234
curl http://192.168.1.11:31234
```

kube-proxy가 적절한 Pod로 트래픽을 전달하며, 노드 장애 시 다른 노드로 접근할 수 있습니다.

### 4.2. Ingress (단일 도메인)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
  namespace: default
spec:
  type: ClusterIP          # Ingress 사용 시 ClusterIP로 충분
  selector:
    app: webapp
  ports:
    - port: 8080
      targetPort: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "traefik"   # HyperCloud는 Traefik 사용
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 8080
```

```bash
# /etc/hosts 또는 DNS에 등록 후 접근
# <Ingress-Controller-IP>  app.example.com
curl http://app.example.com
```

### 4.3. Ingress (Path 기반 다중 서비스 라우팅)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-service-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "traefik"
    traefik.ingress.kubernetes.io/rewrite-target: "/"
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 3000
```

```bash
curl http://app.example.com/api    # → api-service:8080
curl http://app.example.com/web    # → web-service:80
curl http://app.example.com/admin  # → admin-service:3000
```

### 4.4. Ingress (Host 기반 다중 도메인 라우팅)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
  - host: web.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port:
              number: 3000
```

```bash
curl http://api.example.com    # → api-service:8080
curl http://web.example.com    # → web-service:80
curl http://admin.example.com  # → admin-service:3000
```

### 4.5. Ingress with TLS/SSL

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
  namespace: default
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...  # Base64 인코딩된 인증서
  tls.key: LS0tLS1CRUdJTi...  # Base64 인코딩된 개인키
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress-tls
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "traefik"
    traefik.ingress.kubernetes.io/redirect-entry-point: "https"  # HTTP → HTTPS 리다이렉트
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 8080
```

```bash
# TLS 종료는 Ingress Controller(Traefik)에서 처리
curl https://app.example.com

# Self-signed 인증서 생성 (테스트용)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=app.example.com"

kubectl create secret tls tls-secret \
  --cert=tls.crt --key=tls.key -n default
```

### 4.6. 개발/운영 환경 분리 (혼합 사용)

```yaml
# 개발 환경: NodePort로 빠른 테스트
apiVersion: v1
kind: Service
metadata:
  name: dev-app-svc
  namespace: dev
spec:
  type: NodePort
  selector:
    app: dev-app
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 31000
---
# 운영 환경: Ingress + TLS로 도메인 기반 접근
apiVersion: v1
kind: Service
metadata:
  name: prod-app-svc
  namespace: prod
spec:
  type: ClusterIP
  selector:
    app: prod-app
  ports:
    - port: 8080
      targetPort: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prod-app-ingress
  namespace: prod
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: prod-tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prod-app-svc
            port:
              number: 8080
```

```bash
curl http://192.168.1.10:31000   # 개발 환경
curl https://app.example.com     # 운영 환경
```

---

## 5. HyperCloud Traefik 실제 포트 구성

HyperCloud 환경의 Traefik Service는 다음과 같이 여러 포트를 NodePort로 노출합니다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik
  namespace: api-gateway-system
spec:
  type: NodePort
  selector:
    app: traefik
  ports:
  - name: web
    nodePort: 30000
    port: 80
    protocol: TCP
    targetPort: web
  - name: websecure
    nodePort: 30001
    port: 443
    protocol: TCP
    targetPort: websecure
  - name: traefik        # Dashboard
    nodePort: 30900
    port: 9000
    protocol: TCP
    targetPort: traefik
  - name: k8s
    nodePort: 30266
    port: 6443
    protocol: TCP
    targetPort: k8s
  - name: metrics
    nodePort: 31958
    port: 9100
    protocol: TCP
    targetPort: metrics
```

```bash
# Traefik Dashboard 접근
kubectl port-forward -n traefik-system <traefik-pod> 9000:9000
# 브라우저: http://localhost:9000/dashboard/
```

---

## 6. NLB 포함 전체 트래픽 흐름 (실무 아키텍처)

```
┌───────────────────────────────────────────────────────┐
│                        사용자                           │
└─────────────────────────┬─────────────────────────────┘
                          │ DNS 조회
                          │ myapp.example.com → NLB IP
                          ▼
┌───────────────────────────────────────────────────────┐
│         AWS NLB (L4, 52.78.123.45:443)                │
│         Target Group: Worker Nodes :30001             │
└─────────────────────────┬─────────────────────────────┘
                          │ TCP :30001
                          ▼
┌───────────────────────────────────────────────────────┐
│    Worker Node (10.0.1.10:30001) — NodePort Service   │
└─────────────────────────┬─────────────────────────────┘
                          │ kube-proxy (iptables)
                          ▼
┌───────────────────────────────────────────────────────┐
│    Traefik Pod (10.244.1.5:8443)                      │
│    - TLS 종료                                          │
│    - Ingress 규칙 매칭 (host + path)                    │
└─────────────────────────┬─────────────────────────────┘
                          │ host: myapp.example.com / path: /api
                          ▼
┌───────────────────────────────────────────────────────┐
│    Service: my-app-service (ClusterIP 10.100.50.20)   │
└─────────────────────────┬─────────────────────────────┘
                          │ kube-proxy (iptables)
                          ▼
┌───────────────────────────────────────────────────────┐
│    Application Pod (10.244.2.15:8080)                 │
└───────────────────────────────────────────────────────┘
```

이 구조에서 Service가 2개 필요한 이유를 정리하면 다음과 같습니다.

| Service | 타입 | 역할 |
|---------|------|------|
| **Traefik Service** | NodePort | NLB(외부) → Traefik Pod 연결 |
| **Application Service** | ClusterIP | Traefik → 애플리케이션 Pod 연결 |

---

## 7. 트러블슈팅

### 7.1. NodePort 디버깅

```bash
# 1. Service 생성 확인
kubectl get svc -n <namespace>

# 2. Endpoint 생성 확인 (Pod 연결 여부)
kubectl get endpoints <service-name> -n <namespace>

# 3. 노드에서 직접 포트 확인
netstat -tulnp | grep <node-port>

# 4. kube-proxy 로그 확인
kubectl logs -n kube-system <kube-proxy-pod>
```

### 7.2. Ingress 디버깅

```bash
# 1. Ingress Controller Pod 실행 확인
kubectl get pods -n traefik-system

# 2. Ingress 리소스 상태 확인
kubectl describe ingress <ingress-name> -n <namespace>

# 3. Ingress Controller 로그 확인
kubectl logs -n traefik-system <traefik-pod> -f

# 4. Host 헤더 직접 테스트
curl -H "Host: app.example.com" http://<ingress-controller-ip>

# 5. DNS 확인
nslookup app.example.com

# 6. 연결된 Service 확인
kubectl get svc <service-name> -n <namespace>
```

### 7.3. 흔한 실수 7가지

**1. NodePort 범위 초과**

```
❌ nodePort: 25000  (30000-32767 범위 밖)
→ Error: provided port is not in the valid range

✅ nodePort: 31234  (범위 내 포트)
   또는 생략하여 자동 할당
```

**2. Ingress 뒤의 Service를 NodePort로 생성**

```
❌ Ingress 뒤의 Service를 NodePort로 생성
→ 불필요한 포트 낭비, 보안 취약점

✅ Ingress 사용 시 Service는 ClusterIP로 충분
```

**3. Ingress Controller 미설치**

```
❌ Ingress 리소스만 생성
→ Ingress 작동 안 함 (Controller가 없음)

✅ Ingress Controller 설치 확인
   kubectl get pods -n traefik-system
```

**4. Host 헤더 없이 IP로만 접근**

```
❌ curl http://<ingress-controller-ip>
→ 404 Not Found (Host 헤더 불일치)

✅ Host 헤더 포함 또는 DNS 설정
   curl -H "Host: app.example.com" http://<ip>
```

**5. pathType 미지정**

```
❌ path: /api (pathType 생략)
→ Kubernetes 1.18+ 필수 필드, 오류 발생

✅ pathType: Prefix 또는 Exact 명시
```

**6. TLS Secret이 다른 Namespace**

```
❌ Ingress: default namespace
   Secret:  kube-system namespace
→ TLS 인증서 로드 실패

✅ Ingress와 Secret은 반드시 동일 namespace
```

**7. NodePort 방화벽 미개방**

```
❌ NodePort 31234 생성했지만 접근 불가
→ AWS Security Group, 방화벽에서 포트 차단

✅ 노드의 방화벽/Security Group에서 해당 포트 허용
```

---

## 8. 의사결정 가이드

### 언제 NodePort를 사용하나

- 개발/테스트 환경에서 빠른 접근이 필요할 때
- HTTP/HTTPS가 아닌 프로토콜 (TCP/UDP)
- Ingress Controller 설치가 불가능한 환경
- 단순한 외부 노출 (1~2개 서비스)

### 언제 Ingress를 사용하나

- 프로덕션 환경
- 여러 서비스를 하나의 도메인으로 제공
- Path/Host 기반 라우팅 필요
- SSL/TLS 종료 필요
- 표준 HTTP/HTTPS 포트 사용 (80, 443)

### 금융권 환경 고려사항

| 구간 | 권장 방식 |
|------|-----------|
| **외부망** | Ingress + TLS 필수 |
| **내부망** | NodePort로 간단히 노출 가능 |
| **DMZ** | Ingress + WAF 연동 |
| **감사 추적** | Ingress Controller 로그 수집 |

---

## 9. 운영 참고사항

**HyperCloud 환경**: Traefik이 기본 Ingress Controller로 설치됨

**포트 할당 전략**:
- NodePort: 31000번대 개발 / 32000번대 테스트로 구분
- Ingress: 도메인 네이밍 규칙 정의 (dev.app.com, prod.app.com)

**실무 아키텍처 패턴**:

```
외부 사용자 → [AWS NLB] → Traefik (Ingress Controller) → Service → Pod
내부 개발자 → [NodePort] → Service → Pod
```

**Prometheus 모니터링**:
- NodePort: `kube_service_spec_type{type="NodePort"}`
- Ingress: Traefik 메트릭 (`traefik_service_requests_total`)

---

## 참고 자료

- Kubernetes Service 공식 문서: <https://kubernetes.io/docs/concepts/services-networking/service/>
- Kubernetes Ingress 공식 문서: <https://kubernetes.io/docs/concepts/services-networking/ingress/>