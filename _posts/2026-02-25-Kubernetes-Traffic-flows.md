---
title: "[Kubernetes] Ingress와 Service 트래픽 흐름 완전 정리: NLB → Traefik → Pod"
date: 2026-02-25 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, traefik, ingress, service, nlb, nodeport, clusterip, traffic, kube-proxy, tls]
render_with_liquid: false
---

# Kubernetes Ingress와 Service 동작 흐름 완전 정리

## 1. 기본 구성 요소

### 1.1. Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
    protocol: TCP
```

**역할**: Traefik → 애플리케이션 Pod 연결

### 1.2. Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

**역할**: 라우팅 규칙 정의 (Traefik에게 주는 설정 파일)

### 1.3. 기본 동작 흐름

```
사용자 요청
  ↓
http://myapp.example.com
  ↓
Ingress (Traefik)
  ↓
Service (my-app-service:80)
  ↓
Pod (app=my-app, port 8080)
```

---

## 2. NLB를 포함한 전체 아키텍처

### 2.1. 전체 구조

```
사용자
  ↓
DNS (myapp.example.com → NLB IP)
  ↓
AWS NLB (Network Load Balancer)
  ↓
Traefik Ingress Controller (NodePort Service)
  ↓
Ingress 규칙 매칭
  ↓
Service (ClusterIP)
  ↓
Pod
```

### 2.2. Traefik Service (NodePort 타입)

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
  - name: websecure
    port: 443
    targetPort: 8443
    nodePort: 31443
```

**역할**: NLB → Traefik Pod 연결

### 2.3. 상세 패킷 흐름

```
[사용자] curl https://myapp.example.com/api
    ↓
[DNS] myapp.example.com → 52.78.123.45 (NLB)
    ↓
[NLB] 52.78.123.45:443
    ↓ (Target Group)
[Worker Node 1] 10.0.1.10:31443 (NodePort)
    ↓
[Traefik Pod] 10.244.1.5:8443 (Ingress Controller)
    ↓ (Ingress 규칙 매칭: host + path)
[Service] my-app-service:80 (ClusterIP: 10.100.50.20)
    ↓ (kube-proxy iptables)
[Pod] 10.244.2.15:8080 (실제 애플리케이션)
    ↓
[응답] 역순으로 반환
```

### 2.4. NLB vs ALB 비교

| 항목 | NLB (Network LB) | ALB (Application LB) |
|------|------------------|----------------------|
| **OSI 계층** | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
| **성능** | 매우 빠름 (백만 RPS) | 상대적으로 느림 |
| **라우팅** | IP/Port만 | Host, Path, Header 기반 |
| **SSL 종료** | Traefik에서 처리 | ALB에서 처리 가능 |
| **HyperCloud** | 주로 사용 | 선택적 사용 |

---

## 3. Service 2개의 역할

### 3.1. 구성

**Service 1 — Traefik Service (NodePort)**

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
  - name: websecure
    port: 443
    targetPort: 8443
    nodePort: 31443
```

**Service 2 — Application Service (ClusterIP)**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 8080
```

### 3.2. 흐름 요약

```
[NLB]
  ↓
[Service 1: traefik (NodePort 31443)]    ← 외부 진입점
  ↓
[Traefik Pod (Ingress Controller)]       ← L7 라우팅
  ↓ (Ingress 규칙 확인)
[Service 2: my-app-service (ClusterIP 80)] ← 내부 연결
  ↓
[Application Pod (8080)]
```

### 3.3. 왜 Service가 2개 필요한가?

**Service 1 (Traefik Service)**:
- 목적: 외부에서 Traefik Pod에 접근
- 타입: NodePort (또는 LoadBalancer)
- 이유: NLB는 Kubernetes 클러스터 외부에 있으므로 NodePort로 노출 필요

**Service 2 (Application Service)**:
- 목적: Traefik에서 애플리케이션 Pod로 트래픽 전달
- 타입: ClusterIP (내부 통신 전용)
- 이유: Ingress 리소스가 이 Service를 참조하여 Pod로 라우팅

---

## 4. Ingress 리소스의 역할

### 4.1. 핵심 개념

**Ingress = Traefik에게 주는 라우팅 지시서 (설정 파일)**

- Service가 아님 ❌
- 직접 트래픽을 받지 않음 ❌
- Ingress Controller(Traefik)가 읽고 적용하는 규칙 ✅

### 4.2. Ingress vs Ingress Controller

| 항목 | Ingress | Ingress Controller |
|------|---------|--------------------|
| **종류** | Kubernetes 리소스 (설정) | 실제 Pod (프로그램) |
| **역할** | "어디로 보낼지" 규칙 정의 | 규칙을 읽고 실제 라우팅 |
| **형태** | YAML 매니페스트 | Traefik, Nginx 등 |
| **위치** | etcd에 저장 | Worker Node에서 실행 |

### 4.3. Ingress 동작 원리

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: my-app-service  # 이 Service로 보내라!
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
              number: 80
```

**Traefik 라우팅 처리 흐름**:

```
요청: https://myapp.example.com/api/users
  ↓
Traefik: "Ingress 규칙 확인"
  ↓
Traefik: "host = myapp.example.com, path = /api 로 시작"
  ↓
Traefik: "Ingress에 my-app-service:80으로 보내라고 정의됨"
  ↓
Traefik: my-app-service로 프록시
```

### 4.4. Ingress가 필요한 이유

**Ingress 없이 NodePort만 사용하면?**

```
# 서비스마다 다른 NodePort 필요
Service 1: NodePort 31001 (app1.example.com)
Service 2: NodePort 31002 (app2.example.com)
Service 3: NodePort 31003 (app3.example.com)
```

**Ingress 사용하면?**

```
# NodePort는 Traefik 하나만 (30001)
# Ingress로 도메인/Path 기반 라우팅

Ingress:
  app1.example.com       → app1-service
  app2.example.com       → app2-service
  app3.example.com/api   → app3-service
  app3.example.com/admin → admin-service
```

장점: NodePort 하나만 열면 되고, 도메인/Path 기반 라우팅과 SSL 인증서 관리를 Traefik에서 중앙 처리할 수 있습니다.

---

## 5. Traefik Service 포트 분석

### 5.1. 실제 포트 구성

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
  - name: traefik
    nodePort: 30900
    port: 9000
    protocol: TCP
    targetPort: traefik
  - name: metrics
    nodePort: 31958
    port: 9100
    protocol: TCP
    targetPort: metrics
  - name: k8s
    nodePort: 30266
    port: 6443
    protocol: TCP
    targetPort: k8s
```

### 5.2. 포트별 역할

| 포트 이름 | NodePort | Service Port | targetPort | 역할 |
|-----------|----------|--------------|------------|------|
| **web** | 30000 | 80 | web | HTTP 트래픽 (주 진입점) |
| **websecure** | 30001 | 443 | websecure | HTTPS 트래픽 (주 진입점) |
| **traefik** | 30900 | 9000 | traefik | Traefik Dashboard |
| **metrics** | 31958 | 9100 | metrics | Prometheus 메트릭 |
| **k8s** | 30266 | 6443 | k8s | Kubernetes API 프록시 |

### 5.3. NLB Target Group 설정

```
Target Group 1 (HTTP)
- Protocol: TCP
- Port: 30000
- Targets: Worker Node 1~N의 IP

Target Group 2 (HTTPS)
- Protocol: TCP
- Port: 30001
- Targets: Worker Node 1~N의 IP
```

### 5.4. targetPort를 이름으로 쓰는 이유

**Service에서**:

```yaml
ports:
- name: web
  nodePort: 30000
  port: 80
  targetPort: web   # 포트 이름으로 매칭
```

**Traefik Pod에서**:

```yaml
containers:
- name: traefik
  ports:
  - name: web       # Service의 targetPort와 매칭
    containerPort: 8080
```

포트 번호 대신 이름을 사용하면 컨테이너 포트 번호가 바뀌어도 Service를 수정할 필요가 없어 유지보수가 편리합니다.

---

## 6. 실제 패킷 흐름

### 6.1. HTTP 트래픽 (포트 80)

```
[사용자] http://myapp.example.com
  ↓
[DNS] myapp.example.com → NLB IP
  ↓
[NLB] :80 → Worker Node들의 30000 포트로 분산
  ↓
[Worker Node] 10.0.1.10:30000 (NodePort)
  ↓
[Service: traefik] port:80 → targetPort:web
  ↓
[Traefik Pod] web 포트 (8080)
  ↓
[Ingress 규칙 확인]
  ↓
[ClusterIP Service: my-app-service:80]
  ↓
[Application Pod:8080]
```

### 6.2. HTTPS 트래픽 (포트 443) — 실무 주 사용

```
[사용자]
curl https://myapp.example.com/api/users
  ↓
[DNS 조회]
myapp.example.com → 52.78.123.45 (NLB)
  ↓
[NLB]
52.78.123.45:443
  ↓ Target Group (30001)
[Worker Node 1]
10.0.1.10:30001
  ↓ iptables (kube-proxy)
[Service: traefik]
port:443 → targetPort:websecure
  ↓
[Traefik Pod]
10.244.1.5:8443 (websecure 포트)
  ↓ TLS 종료 (인증서 검증)
  ↓ Ingress 규칙 확인
      - Host: myapp.example.com ✅
      - Path: /api              ✅
      - Backend: my-app-service:80
  ↓
[Service: my-app-service]
ClusterIP 10.100.50.20:80
  ↓ kube-proxy
[Application Pod]
10.244.2.15:8080
  ↓
[응답] 역순으로 반환
```

### 6.3. 관리 포트 접근

**Traefik Dashboard**:

```bash
# 외부에서 접근 (NodePort)
http://<worker-node-ip>:30900/dashboard/

# 또는 Port Forward
kubectl port-forward -n api-gateway-system svc/traefik 9000:9000
# 브라우저: http://localhost:9000/dashboard/
```

**Prometheus 메트릭**:

```bash
curl http://<worker-node-ip>:31958/metrics
```

---

## 7. 전체 구조 다이어그램

```
┌─────────────────────────────────────────────────────────┐
│                         사용자                            │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ DNS 조회
                     │ myapp.example.com → 52.78.123.45
                     ▼
┌─────────────────────────────────────────────────────────┐
│              AWS NLB (52.78.123.45)                     │
│  Target Group: Worker Nodes :30001 (HTTPS)              │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ TCP :30001
                     ▼
┌─────────────────────────────────────────────────────────┐
│           Worker Node (10.0.1.10:30001)                 │
│                  NodePort Service                        │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ kube-proxy (iptables)
                     ▼
┌─────────────────────────────────────────────────────────┐
│   Service: traefik (NodePort)                           │
│   Selector: app=traefik                                 │
│   Port: 443 → targetPort: websecure                     │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│   Traefik Pod (10.244.1.5:8443)                         │
│   - TLS 종료                                             │
│   - Ingress 리소스 읽기 (etcd에서)                        │
│   - 라우팅 규칙 적용                                       │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ Ingress 규칙:
                     │ host: myapp.example.com
                     │ path: /api
                     │ → my-app-service:80
                     ▼
┌─────────────────────────────────────────────────────────┐
│   Service: my-app-service (ClusterIP)                   │
│   Selector: app=my-app                                  │
│   Port: 80 → targetPort: 8080                           │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ kube-proxy (iptables)
                     ▼
┌─────────────────────────────────────────────────────────┐
│   Application Pod (10.244.2.15:8080)                    │
│   Label: app=my-app                                     │
│   실제 애플리케이션 실행                                    │
└─────────────────────────────────────────────────────────┘
```

---

## 8. 확인 명령어 모음

### Service 확인

```bash
# Traefik Service 확인 (NLB IP 확인)
kubectl get svc -n api-gateway-system traefik

# 애플리케이션 Service 확인
kubectl get svc -n default my-app-service

# Service 상세 정보
kubectl describe svc -n api-gateway-system traefik
```

### Ingress 확인

```bash
# Ingress 목록
kubectl get ingress --all-namespaces

# Ingress 상세 정보 (Host, Path, Backend 확인)
kubectl describe ingress my-app-ingress -n default

# Ingress YAML 확인
kubectl get ingress my-app-ingress -n default -o yaml
```

### Pod 확인

```bash
# Traefik Pod 확인
kubectl get pods -n api-gateway-system -l app=traefik

# 애플리케이션 Pod 확인
kubectl get pods -n default -l app=my-app

# Traefik Pod 로그
kubectl logs -n api-gateway-system <traefik-pod> -f

# Traefik Pod의 실제 리스닝 포트 확인
kubectl exec -it -n api-gateway-system <traefik-pod> -- netstat -tlnp
```

### Endpoint 확인

```bash
# Service Endpoint 확인 (Pod 연결 상태)
kubectl get endpoints my-app-service -n default

# Traefik Service Endpoint
kubectl get endpoints traefik -n api-gateway-system
```

### 트러블슈팅 체크리스트

```bash
# 1. DNS → NLB 연결 확인
nslookup myapp.example.com

# 2. NLB → Node 연결
# AWS Console에서 Target Group Health 확인

# 3. Node → Traefik Service 확인
kubectl get svc -n api-gateway-system traefik

# 4. Traefik → Service (Ingress 규칙 확인)
kubectl describe ingress my-app-ingress -n default

# 5. Service → Pod (Endpoint 확인)
kubectl get endpoints my-app-service -n default

# 6. Traefik이 Ingress를 인식했는지 확인
kubectl logs -n api-gateway-system <traefik-pod> | grep -i ingress

# 7. 실제 연결 테스트
curl -v https://myapp.example.com
```

---

## 9. 핵심 정리

각 구성 요소의 역할을 한 줄로 정리하면 아래와 같습니다.

| 구성 요소 | 역할 |
|-----------|------|
| **NLB** | 외부에서 Kubernetes 클러스터로 진입하는 L4 로드밸런서 |
| **NodePort Service (traefik)** | NLB와 Traefik Pod를 연결 (30001 포트) |
| **Traefik Pod** | Ingress Controller로 실제 L7 라우팅 수행 |
| **Ingress 리소스** | Traefik에게 주는 라우팅 규칙 (설정 파일, etcd 저장) |
| **ClusterIP Service** | Traefik과 애플리케이션 Pod 연결 (내부 통신) |
| **Application Pod** | 실제 애플리케이션이 실행되는 컨테이너 |

**데이터 흐름 한 줄 요약**:

```
NLB → NodePort(traefik) → Traefik Pod → (Ingress 규칙 확인) → ClusterIP Service → Application Pod
```

---

## 참고 자료

- Kubernetes Service 공식 문서: <https://kubernetes.io/docs/concepts/services-networking/service/>
- Kubernetes Ingress 공식 문서: <https://kubernetes.io/docs/concepts/services-networking/ingress/>