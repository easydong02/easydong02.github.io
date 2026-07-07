---
title: "[Kubernetes] Secret"
date: 2026-03-11 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, secret, security, tls, rbac]
render_with_liquid: false
mermaid: true
---

## 📌 들어가며

이번 글에서는 파드가 쓰는 민감 정보(비밀번호·인증서·토큰)를 안전하게 다루는 **Secret**을 정리한다. 다양한 Secret 타입, 파드에 주입하는 방법, 그리고 반드시 지켜야 할 보안 주의사항까지 살펴본다.

> **Secret이란?** 비밀번호·인증서·토큰 같은 **민감한 정보를 저장·관리**하는 쿠버네티스 리소스. 코드/이미지에 하드코딩하지 않고 분리하며, **환경 변수** 또는 **Volume**으로 파드에 주입한다. 데이터는 `base64`로 인코딩되고(암호화 아님), etcd 부하를 막기 위해 **1MB로 제한**된다.

---

## 1. Secret 타입 총정리

`type` 필드는 **"이 봉투에 어떤 형식의 데이터가 들어있는지"**를 시스템에 알려준다.

| 타입 | 비유 | 주요 Key | 비고 |
|------|------|----------|------|
| **Opaque** | 빈 봉투 | (사용자 정의) | 가장 많이 씀(기본) |
| **kubernetes.io/tls** | 신분증 세트 | `tls.crt`, `tls.key` | 인증서 만료 관리 |
| **service-account-token** | 사원증 | `token`, `ca.crt` | 파드에 자동 마운트 |
| **kubernetes.io/basic-auth** | 로그인 메모 | `username`, `password` | Ingress 인증 |
| **dockerconfigjson** | 창고 열쇠 | `.dockerconfigjson` | Private 이미지 pull |
| **helm.sh/release.v1** | 설치 이력서 | (Helm 내부) | **건드리지 말 것** 🚫 |

### Opaque (일반형)

정해진 형식이 없는 임의의 Key-Value. DB 비밀번호·API Key 등 대부분의 경우.

```yaml
apiVersion: v1
kind: Secret
type: Opaque
data:
  db-pass: MWYyaZD...   # base64
  api-key: dXNlcnB...
```

### kubernetes.io/tls (HTTPS 인증서)

Ingress·웹 서버의 TLS/SSL 인증서 표준 규격. `tls.crt`(공개 인증서)·`tls.key`(개인키)가 필수.

```bash
kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key -n default
```

### dockerconfigjson (이미지 풀 시크릿)

Private 레지스트리 로그인 정보. 파드의 `imagePullSecrets`에 사용.

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-server=hyperregistry.example.com \
  --docker-username=myuser --docker-password=mypassword -n default
```

```yaml
spec:
  imagePullSecrets:
  - name: my-registry-secret
  containers:
  - name: app
    image: hyperregistry.example.com/myapp:1.0
```

> ⚠️ **`helm.sh/release.v1`은 절대 수동 삭제·수정 금지.** Helm이 배포 상태를 관리하는 전용 시크릿이라, 건드리면 Helm이 배포된 앱을 인식하지 못하게 된다.

---

## 2. Secret 생성 & 조회

```bash
# 명령어 생성(Opaque)
kubectl create secret generic my-secret \
  --from-literal=db-password=mysecretpassword --from-literal=api-key=abc123 -n default

# 파일로부터
kubectl create secret generic my-secret --from-file=config.json -n default

# YAML은 base64 인코딩 필요
echo -n 'mysecretpassword' | base64   # bXlzZWNyZXRwYXNzd29yZA==
```

```bash
# 조회 & 디코딩
kubectl get secret my-secret -n default -o yaml
kubectl get secret my-secret -n default -o jsonpath='{.data.db-password}' | base64 -d
```

---

## 3. 파드에서 사용

### 환경 변수로 주입

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_PASSWORD            # 단일 키
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: db-password
    envFrom:                       # 전체 주입
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
      items:
      - key: db-password
        path: db-password.txt      # /etc/secrets/db-password.txt
```

> 💡 **인증서·설정 파일은 Volume, 짧은 값은 환경 변수**가 편하다. TLS 인증서처럼 파일로 존재해야 하는 것은 Volume 마운트, DB 비밀번호 같은 단순 값은 `secretKeyRef`로 주입한다.

---

## 4. 보안 주의사항 (필수)

> ⚠️ **① Base64는 암호화가 아니다.** `kubectl get secret -o yaml`이면 누구나 디코딩해 원본을 본다.
> ```bash
> echo "bXlzZWNyZXRwYXNzd29yZA==" | base64 -d   # mysecretpassword
> ```

> ⚠️ **② RBAC로 조회 권한 제한.** 아무나 Secret을 `get/list`하지 못하도록 Role을 엄격히 건다.
> ```yaml
> kind: Role
> rules:
> - apiGroups: [""]
>   resources: ["secrets"]
>   resourceNames: ["my-secret"]   # 특정 Secret만
>   verbs: ["get"]
> ```

> ⚠️ **③ Git에 올리지 말 것.** Secret YAML을 그대로 커밋하면 유출된다. **Sealed Secrets**·**External Secrets** 사용을 권장하고, `.gitignore`에 `*-secret.yaml`을 추가한다.

---

## 📝 정리

```
Secret
├─ 개념   민감 정보 저장(base64·1MB 제한)
├─ 타입   Opaque(기본)/tls/dockerconfigjson/basic-auth...
├─ 주입   env(secretKeyRef) / Volume(마운트)
└─ 보안   base64≠암호화 · RBAC 제한 · Git 금지
```

| 개념 | 한 줄 정의 |
|------|------|
| **Secret** | 민감 정보 관리 리소스 |
| **Opaque / tls** | 범용 / 인증서 타입 |
| **base64** | 인코딩(암호화 아님) |

Secret의 핵심은 **민감 정보를 코드에서 분리**하는 것이지만, **base64는 암호화가 아니라는 점**을 반드시 기억해야 한다. RBAC로 접근을 좁히고 Git 유출을 막는 실무 습관이 Secret 자체보다 중요하다.

---

## 🔗 참고

- [공식 문서 - Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [공식 문서 - Secret Types](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)
