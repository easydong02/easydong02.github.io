---
title: "[Kubernetes] Secret"
date: 2026-03-11 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, secret, security, tls, rbac]
render_with_liquid: false
---

## 📌 개요

**한 줄 요약**: 파드가 필요로 하는 민감한 정보(비밀번호, 인증서, 토큰 등)를 안전하게 저장하고 관리하는 쿠버네티스 리소스

**왜 사용하는가**:
- **보안**: 애플리케이션 코드나 이미지 안에 비밀번호를 하드코딩하지 않기 위함
- **유연성**: 개발/운영 환경에 따라 설정값만 바꿔 끼울 수 있어 이미지를 다시 빌드할 필요 없음
- **분리**: "애플리케이션 로직"과 "보안 설정"을 분리하여 관리 가능

**특징**:
- **저장 방식**: 데이터는 `base64`로 인코딩되어 저장 (암호화가 아님에 주의!)
- **사용 방식**: 파드 내부에 **환경 변수**로 주입하거나, **Volume Mount** 형태로 마운트하여 사용
- **용량 제한**: etcd의 부하를 막기 위해 1MB로 제한

---

## 🗂 Secret Type 총정리

Secret의 `type` 필드는 **"이 봉투 안에 어떤 형식의 데이터가 들어있는지"** 를 시스템에 알려주는 역할을 합니다.

### ① Opaque (일반형)

가장 기본적이고 범용적인 타입. "불투명"이라는 뜻으로, 사용자가 정의한 임의의 Key-Value 데이터를 담습니다.

- **용도**: DB 비밀번호, API Key, 설정 파일 등 정해진 형식이 없는 대부분의 경우
- **필수 키**: 없음 (사용자 마음대로 정의)

```yaml
apiVersion: v1
kind: Secret
type: Opaque
data:
  db-pass: MWYyaZD...   # base64 인코딩 값
  api-key: dXNlcnB...
```

---

### ② kubernetes.io/service-account-token (파드 인증용)

파드가 쿠버네티스 API 서버와 통신할 때 신원을 증명하기 위한 **JWT 토큰**을 담고 있습니다.

- **용도**: ServiceAccount 생성 시 자동으로 만들어지거나, API 접근용 토큰이 필요할 때 사용
- **필수 키**:
  - `ca.crt`: API 서버 인증서
  - `token`: 실제 Bearer Token (JWT)
  - `namespace`: 해당 계정의 네임스페이스

---

### ③ kubernetes.io/tls (HTTPS 인증서용)

웹 서버의 HTTPS 통신이나 내부 보안 통신을 위한 **TLS/SSL 인증서**를 저장하는 표준 규격입니다.

- **용도**: Ingress 컨트롤러의 TLS 설정, 웹 서버 파드의 인증서 마운트
- **필수 키**:
  - `tls.crt`: 공개 인증서 (Certificate)
  - `tls.key`: 개인키 (Private Key)

```bash
# TLS Secret 생성
kubectl create secret tls my-tls-secret \
  --cert=tls.crt \
  --key=tls.key \
  -n default
```

---

### ④ kubernetes.io/basic-auth (기본 인증용)

HTTP Basic Authentication을 위한 자격 증명입니다.

- **용도**: Ingress에 간단한 로그인(접근 제어) 기능을 붙일 때 사용
- **필수 키**:
  - `username`: 사용자 아이디
  - `password`: 비밀번호

---

### ⑤ kubernetes.io/dockerconfigjson (이미지 풀 시크릿)

Private 컨테이너 레지스트리에서 이미지를 다운로드할 때 필요한 로그인 정보입니다.

- **용도**: 파드 설정의 `imagePullSecrets` 항목에 사용
- **필수 키**: `.dockerconfigjson`

```bash
# Docker Registry Secret 생성
kubectl create secret docker-registry my-registry-secret \
  --docker-server=hyperregistry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  -n default
```

```yaml
# 파드에서 사용
spec:
  imagePullSecrets:
  - name: my-registry-secret
  containers:
  - name: app
    image: hyperregistry.example.com/myapp:1.0
```

---

### ⑥ helm.sh/release.v1 (Helm 배포 이력)

쿠버네티스 패키지 매니저인 **Helm**이 배포 상태를 관리하기 위해 사용하는 전용 시크릿입니다.

- **용도**: Helm 차트의 버전, values.yaml, 배포 상태 등을 저장
- **⚠️ 주의**: **절대 수동으로 삭제하거나 수정하면 안 됩니다.** Helm이 배포된 앱을 인식하지 못하게 됩니다.
- **특징**: 데이터가 압축되어 저장됩니다.

---

## 📋 타입 요약 테이블

| 타입 | 역할 (비유) | 주요 Key | 비고 |
|------|-------------|----------|------|
| **Opaque** | 빈 봉투 | (사용자 정의) | 가장 많이 사용됨 (Default) |
| **kubernetes.io/tls** | 신분증 세트 | `tls.crt`, `tls.key` | 인증서 만료 관리 필요 |
| **kubernetes.io/service-account-token** | 사원증 | `token`, `ca.crt` | 파드 내부에 자동 마운트됨 |
| **kubernetes.io/basic-auth** | 로그인 메모 | `username`, `password` | Ingress 인증에 주로 사용 |
| **kubernetes.io/dockerconfigjson** | 창고 열쇠 | `.dockerconfigjson` | Private 이미지 받을 때 필수 |
| **helm.sh/release.v1** | 설치 이력서 | (Helm 내부 데이터) | **건드리지 말 것** 🚫 |

---

## 💻 기본 사용법

### Secret 생성

```bash
# 명령어로 생성 (Opaque)
kubectl create secret generic my-secret \
  --from-literal=db-password=mysecretpassword \
  --from-literal=api-key=abc123 \
  -n default

# 파일로부터 생성
kubectl create secret generic my-secret \
  --from-file=config.json \
  -n default

# YAML로 생성 (base64 인코딩 필요)
echo -n 'mysecretpassword' | base64
# bXlzZWNyZXRwYXNzd29yZA==
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: default
type: Opaque
data:
  db-password: bXlzZWNyZXRwYXNzd29yZA==  # base64 인코딩 값
  api-key: YWJjMTIz
```

### Secret 조회

```bash
# Secret 목록
kubectl get secret -n default

# Secret 상세 (인코딩된 값 포함)
kubectl get secret my-secret -n default -o yaml

# 특정 키 값 디코딩
kubectl get secret my-secret -n default \
  -o jsonpath='{.data.db-password}' | base64 -d

# 모든 키-값 디코딩
kubectl get secret my-secret -n default -o go-template='
{{range $k, $v := .data}}{{$k}}: {{$v | base64decode}}
{{end}}'
```

---

## 🔌 파드에서 Secret 사용

### 환경 변수로 주입

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        env:
        # 단일 키 주입
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: db-password
        # Secret 전체 주입
        envFrom:
        - secretRef:
            name: my-secret
```

### Volume으로 마운트

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
      # 특정 키만 마운트
      items:
      - key: db-password
        path: db-password.txt
```

```bash
# 마운트 확인
# /etc/secrets/db-password.txt 파일로 접근 가능
kubectl exec -it <pod-name> -- cat /etc/secrets/db-password.txt
```

---

## ⚠️ 보안 주의사항

**1. Base64는 암호화가 아니다**
`kubectl get secret -o yaml`로 조회하면 누구나 디코딩하여 원본을 볼 수 있습니다.
```bash
# 누구나 이렇게 디코딩 가능
echo "bXlzZWNyZXRwYXNzd29yZA==" | base64 -d
# mysecretpassword
```

**2. RBAC 설정 필수**
아무나 Secret을 조회(`get`, `list`, `watch`)할 수 없도록 Role을 엄격하게 제한해야 합니다.
```yaml
# Secret 조회 권한을 특정 ServiceAccount에만 부여
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["my-secret"]  # 특정 Secret만 허용
  verbs: ["get"]
```

**3. Git에 올리지 말 것**
Secret YAML 파일을 그대로 GitHub에 올리면 유출 사고가 발생합니다. **Sealed Secrets**나 **External Secrets** 같은 도구 사용을 권장합니다.

```bash
# .gitignore에 추가
*-secret.yaml
*-secrets.yaml
```

---

## 🔗 관련 문서

- [공식 문서 - Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [공식 문서 - Secret Types](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)