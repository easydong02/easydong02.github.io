---
title: "[Kubernetes] cert-manager 완전 정리: TLS 인증서 자동 발급과 갱신"
date: 2026-02-25 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, cert-manager, tls, certificate, lets-encrypt, acme, ingress, issuer, clusterissuer, private-ca]
render_with_liquid: false
---

# cert-manager

## 1. 개요

**한 줄 요약**: Kubernetes에서 TLS 인증서를 자동으로 발급, 갱신, 관리하는 인증서 관리 시스템

**언제 사용하나**:
- Ingress에 HTTPS/TLS 적용 시 인증서 자동 발급
- Let's Encrypt로 무료 SSL 인증서 자동 갱신
- 내부 Private CA로 자체 서명 인증서 발급
- 인증서 만료 전 자동 갱신으로 장애 방지

### 인증서 발급 전체 흐름

```
Certificate 리소스 생성
  → Issuer/ClusterIssuer 참조
  → ACME Challenge 수행
  → 인증서 발급
  → Secret 저장 (tls.crt, tls.key)
  → Ingress에서 Secret 참조하여 HTTPS 적용
```

---

## 2. 핵심 개념

### 2.1. 주요 구성 요소

| 리소스 | 역할 |
|--------|------|
| **Certificate** | 인증서 요청 리소스 — 어떤 도메인의 인증서가 필요한지 정의 |
| **Issuer** | 네임스페이스 범위의 인증서 발급자 |
| **ClusterIssuer** | 클러스터 전체 범위의 인증서 발급자 |
| **Secret** | 발급된 인증서 저장 (`tls.crt`, `tls.key`) |
| **CertificateRequest** | Certificate → Issuer 간 인증서 요청 중간 리소스 |
| **Challenge** | ACME 프로토콜의 도메인 소유 증명 방식 |

### 2.2. Issuer vs ClusterIssuer

| 항목 | Issuer | ClusterIssuer |
|------|--------|---------------|
| **범위** | 특정 네임스페이스 | 클러스터 전체 |
| **사용 시기** | 네임스페이스별 독립 관리 | 여러 네임스페이스에서 공통 사용 |
| **예시** | 개발팀별 Let's Encrypt 계정 | 회사 공통 CA 인증서 |

### 2.3. ACME Challenge 방식 비교

| 방식 | 설명 | 장점 | 단점 | 사용 시기 |
|------|------|------|------|-----------|
| **HTTP-01** | `http://<domain>/.well-known/acme-challenge/` 경로에 파일 생성 | 간단, DNS 설정 불필요 | 80 포트 필요, 와일드카드 불가 | 단일 도메인 |
| **DNS-01** | DNS TXT 레코드 추가로 도메인 소유 증명 | 와일드카드 가능, 80/443 불필요 | DNS 제공자 API 필요 | 와일드카드 인증서 |

### 2.4. 주요 특징

- **자동 갱신**: 인증서 만료 30일 전 자동 갱신 시도, 실패 시 매일 재시도
- **다양한 Issuer 지원**: Let's Encrypt, Vault, Venafi, Self-signed
- **선언적 관리**: YAML로 인증서 요청 정의

---

## 3. 기본 명령어

```bash
# cert-manager Pod 3개 실행 확인
# (cert-manager, cert-manager-webhook, cert-manager-cainjector)
kubectl get pods -n cert-manager

# ClusterIssuer / Issuer 확인
kubectl get clusterissuer
kubectl get issuer -n <namespace>

# Certificate 확인 (READY 상태 확인)
kubectl get certificate -n <namespace>

# Certificate 상세 정보 (Events에서 에러 확인)
kubectl describe certificate <cert-name> -n <namespace>

# 인증서 발급 진행 상태 확인
kubectl get certificaterequest -n <namespace>

# ACME Challenge 확인 (발급 중일 때)
kubectl get challenge -n <namespace>

# 발급된 인증서 Secret 확인
kubectl get secret <tls-secret-name> -n <namespace>

# 인증서 내용 확인 (만료일 등)
kubectl get secret <tls-secret-name> -n <namespace> \
  -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text -noout

# cert-manager 로그 확인
kubectl logs -n cert-manager deploy/cert-manager -f
```

**Certificate 상태 출력 예시**:

```
NAME                    READY   SECRET           AGE
app-example-com-tls     True    app-tls-secret   5d

# describe 출력
Status:
  Conditions:
    Status:      True
    Type:        Ready
  Not After:     2024-03-06T12:00:00Z   ← 만료일
  Not Before:    2024-12-06T12:00:00Z
  Renewal Time:  2024-02-05T12:00:00Z   ← 갱신 시작 시간
```

---

## 4. 실무 패턴

### 4.1. cert-manager 설치

```bash
# Helm으로 설치 (권장)
helm repo add jetstack https://charts.jetstack.io
helm repo update

# CRD 먼저 설치
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.crds.yaml

# cert-manager 설치
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.13.0

# 설치 확인
kubectl get pods -n cert-manager
```

### 4.2. Let's Encrypt ClusterIssuer

**프로덕션용**:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik  # HyperCloud 환경
```

**테스트용 Staging** (Rate Limit 없음, 브라우저 신뢰 X):

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: traefik
```

> **Staging 먼저 테스트 필수**: Production은 주당 50개/도메인 Rate Limit이 있습니다.

### 4.3. Ingress에 자동 인증서 발급 (annotation 방식)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # 자동 Certificate 생성
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret   # 인증서가 저장될 Secret 이름
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 8080
```

**동작 과정**:

```
1. Ingress 생성 → cert-manager가 annotation 감지
2. Certificate 리소스 자동 생성
3. Let's Encrypt에 인증서 요청
4. HTTP-01 Challenge 수행 (임시 Ingress 생성)
5. 인증서 발급 → app-tls-secret에 저장
6. Ingress가 Secret 참조하여 HTTPS 적용
```

### 4.4. Certificate 리소스 직접 생성

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-certificate
  namespace: default
spec:
  secretName: app-tls-secret       # 인증서가 저장될 Secret 이름
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - app.example.com
  - www.app.example.com
  duration: 2160h                  # 90일 (Let's Encrypt 기본)
  renewBefore: 720h                # 만료 30일 전부터 갱신 시도
```

```bash
# 적용 후 확인
kubectl get certificate app-certificate -n default
# READY: True → 발급 완료

kubectl get secret app-tls-secret -n default
```

### 4.5. 와일드카드 인증서 (DNS-01 Challenge)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dns
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-dns
    solvers:
    - dns01:
        route53:
          region: ap-northeast-2
          accessKeyID: AKIAIOSFODNN7EXAMPLE
          secretAccessKeySecretRef:
            name: route53-credentials
            key: secret-access-key
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-certificate
  namespace: default
spec:
  secretName: wildcard-tls-secret
  issuerRef:
    name: letsencrypt-dns
    kind: ClusterIssuer
  dnsNames:
  - "*.example.com"   # 와일드카드
  - "example.com"     # 루트 도메인
```

```bash
# Route53 자격 증명 Secret 생성
kubectl create secret generic route53-credentials \
  --from-literal=secret-access-key=<AWS_SECRET_ACCESS_KEY> \
  -n cert-manager
```

### 4.6. Self-signed 인증서 (개발/테스트)

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: dev-certificate
  namespace: default
spec:
  secretName: dev-tls-secret
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
  dnsNames:
  - dev.example.local
  - "*.dev.example.local"
```

외부 CA 없이 즉시 발급되지만, 브라우저에서 "신뢰할 수 없는 인증서" 경고가 발생하므로 개발 환경에서만 사용합니다.

### 4.7. 내부 Private CA (금융권 폐쇄망 환경)

```yaml
# 1. Self-signed Root CA 생성
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: my-ca
  namespace: cert-manager
spec:
  secretName: my-ca-secret
  isCA: true                  # CA 인증서로 설정
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
  commonName: "My Internal CA"
  dnsNames:
  - "My Internal CA"
---
# 2. Private CA를 Issuer로 등록
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: my-ca-issuer
spec:
  ca:
    secretName: my-ca-secret
---
# 3. Private CA로 내부 서비스 인증서 발급
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: internal-app-cert
  namespace: default
spec:
  secretName: internal-app-tls
  issuerRef:
    name: my-ca-issuer
    kind: ClusterIssuer
  dnsNames:
  - internal-app.company.local
```

**금융권 폐쇄망 보안 정책**:
- Let's Encrypt 사용 불가 시 내부 CA 구축
- 인증서 만료 알림 설정 (Prometheus Alert)
- 인증서 Secret 접근 권한 최소화 (RBAC)

---

## 5. 트러블슈팅

### 5.1. 실무 디버깅 순서

```bash
# Step 1: Certificate 상태 확인
kubectl get certificate app-certificate -n default
# READY: False → 문제 있음

# Step 2: Events 확인 (에러 메시지 확인)
kubectl describe certificate app-certificate -n default
# Events:
#   Warning  Failed  Issuer not found

# Step 3: ClusterIssuer 상태 확인
kubectl get clusterissuer letsencrypt-prod
# READY: True 확인

# Step 4: CertificateRequest 확인
kubectl get certificaterequest -n default
kubectl describe certificaterequest <n> -n default

# Step 5: Challenge 확인 (발급 진행 중)
kubectl get challenge -n default
# State: pending → 진행 중
# State: valid   → 성공

# Step 6: 임시 Ingress 확인 (HTTP-01)
kubectl get ingress -n default
# cm-acme-http-solver-xxx 임시 Ingress 생성 여부 확인

# Step 7: 도메인 접근 테스트
curl -v http://app.example.com/.well-known/acme-challenge/

# Step 8: cert-manager 로그
kubectl logs -n cert-manager deploy/cert-manager -f
```

### 5.2. 추가 디버깅 명령어

```bash
# ACME Challenge 수동 테스트 (HTTP-01)
curl -v http://app.example.com/.well-known/acme-challenge/test
# 200 OK 응답 확인

# 인증서 재발급 강제 실행
kubectl delete certificaterequest <request-name> -n <namespace>
kubectl delete secret <tls-secret-name> -n <namespace>
# Certificate가 자동으로 재요청

# Let's Encrypt Rate Limit 확인
# https://crt.sh/?q=example.com 에서 최근 발급 내역 조회

# DNS-01 Challenge TXT 레코드 확인
dig app.example.com TXT
# _acme-challenge.app.example.com TXT 레코드 확인

# cert-manager webhook 확인
kubectl get validatingwebhookconfigurations
kubectl get mutatingwebhookconfigurations
# cert-manager-webhook이 있어야 함
```

### 5.3. 흔한 실수 7가지

**1. ClusterIssuer 생성 전 Certificate 생성**

```
❌ ClusterIssuer: letsencrypt-prod 없음
   Certificate에서 참조
→ Error: issuer not found

✅ ClusterIssuer 먼저 생성 후 Certificate 생성
```

**2. HTTP-01 Challenge 실패 (80 포트 접근 불가)**

```
❌ Ingress Controller가 80 포트 리스닝 안 함
   방화벽에서 80 포트 차단
→ Challenge 실패, 인증서 발급 안 됨

✅ 80 포트 외부 접근 가능하도록 설정
   또는 DNS-01 Challenge로 변경
```

**3. Let's Encrypt Rate Limit 초과**

```
❌ 동일 도메인으로 1시간에 5회 이상 요청
→ Error: too many failed authorizations recently

✅ Staging 환경에서 먼저 테스트
   letsencrypt-staging 사용 후 prod로 전환
```

**4. Ingress annotation 오타**

```
❌ cert-manager.io/issuer: "letsencrypt-prod"
   (ClusterIssuer인데 cluster-issuer 대신 issuer 사용)

✅ cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

**5. 도메인 DNS가 Ingress를 가리키지 않음**

```
❌ app.example.com → DNS 미설정 또는 잘못된 IP
→ HTTP-01 Challenge 실패

✅ 도메인 DNS A 레코드 확인
   nslookup app.example.com
   # Ingress Controller IP와 일치해야 함
```

**6. 인증서 갱신 실패**

```
❌ 갱신 시도 중 Challenge 실패
   cert-manager Pod 장애
→ HTTPS 접속 불가

✅ renewBefore: 720h (30일) 권장
   cert-manager 로그 모니터링
```

**7. 여러 Ingress가 동일 도메인 사용**

```
❌ Ingress A: app.example.com
   Ingress B: app.example.com
→ 인증서 충돌, Challenge 실패

✅ 하나의 Ingress로 통합 또는 다른 도메인 사용
```

---

## 6. 운영 참고사항

### 자동 갱신 타이밍

- Let's Encrypt: 90일 유효, **30일 전부터** 갱신 시도
- `renewBefore: 720h` (30일) 권장
- 갱신 실패 시 매일 재시도

### Staging vs Production

| 항목 | Staging | Production |
|------|---------|------------|
| **용도** | 테스트 | 실서비스 |
| **Rate Limit** | 없음 | 주당 50개/도메인 |
| **브라우저 신뢰** | X | O |

### 와일드카드 제약

- HTTP-01 Challenge는 와일드카드 불가
- DNS-01 Challenge만 와일드카드 지원 (DNS 제공자 API 키 필요)

### 인증서 만료 모니터링 (Prometheus Alert)

```promql
# 7일 이내 만료 예정 인증서 알림
certmanager_certificate_expiration_timestamp_seconds - time() < 604800
```

### 인증서 백업

```bash
kubectl get secret app-tls-secret -n default -o yaml > backup-$(date +%Y%m%d).yaml
```

### cert-manager 업그레이드 시

CRD를 먼저 업그레이드해야 하며, 기존 Certificate 리소스는 자동 마이그레이션되어 유지됩니다.

### HyperCloud 환경 참고

- cert-manager는 HyperCloud 기본 포함 패키지가 아니므로 **별도 설치 필요**
- K8s 1.21 환경: cert-manager **v1.5+** 권장
- API 버전: `cert-manager.io/v1` 사용 (v1.0+ 기준)

---

## 참고 자료

- cert-manager 공식 문서: <https://cert-manager.io/docs/>
- Let's Encrypt 공식 문서: <https://letsencrypt.org/docs/>