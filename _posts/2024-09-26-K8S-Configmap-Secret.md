---
title: "[Kubernetes] ConfigMap과 Secret"
date: 2024-09-26 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, configmap,secret]
render_with_liquid: false
---

# Kubernetes Pod 환경 구성을 위한 오브젝트 - ConfigMap과 Secret

Kubernetes에서 애플리케이션을 배포할 때, 각 Pod는 다양한 환경 설정이 필요합니다. 이때, **ConfigMap**과 **Secret**은 Pod의 환경 변수를 효율적으로 관리하고, 보안 정보나 설정 값을 저장 및 제공하는 데 중요한 역할을 합니다. 이 포스팅에서는 **ConfigMap**과 **Secret**을 사용하여 Pod 환경을 구성하는 방법에 대해 설명하고, 실습 예시를 통해 구체적인 활용법을 살펴보겠습니다.

## 1. ConfigMap과 Secret 이해

### 1.1 ConfigMap

**ConfigMap**은 설정 데이터를 키-값 형태로 저장하는 Kubernetes 오브젝트입니다. 이를 통해 애플리케이션 코드와 환경 설정을 분리하여 관리할 수 있습니다. ConfigMap은 데이터베이스 연결 정보, API 키, 설정 파일 등 애플리케이션에서 필요한 다양한 설정을 저장할 수 있습니다.

### ConfigMap 예시

```yaml
yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  DATABASE_URL: postgres://user:password@db:5432/mydatabase
```

이 ConfigMap은 `APP_ENV`와 `DATABASE_URL`이라는 환경 변수를 저장하고 있습니다.

### 1.2 Secret

**Secret**은 민감한 데이터를 저장하기 위한 Kubernetes 오브젝트로, 보안에 민감한 정보인 비밀번호, 토큰, API 키 등을 저장하는 데 사용됩니다. ConfigMap과 비슷하지만, Secret은 base64로 인코딩된 데이터를 저장하여 좀 더 안전하게 민감한 정보를 다룰 수 있도록 합니다.

### Secret 예시

```yaml
yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64로 인코딩된 "username"
  password: cGFzc3dvcmQ=  # base64로 인코딩된 "password"
```

이 Secret은 `username`과 `password`를 저장하고 있으며, base64로 인코딩된 값을 담고 있습니다.

## 2. ConfigMap 활용

### 2.1 환경 변수를 통한 ConfigMap 사용

ConfigMap을 Pod의 환경 변수로 사용하려면 `envFrom` 필드를 사용합니다. 다음은 ConfigMap을 참조하여 Pod를 생성하는 예시입니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: myapp-container
      image: nginx:1.25.3-alpine
      envFrom:
        - configMapRef:
            name: app-config
```

이 Pod는 `app-config` ConfigMap을 환경 변수로 사용하여 실행됩니다.

### 2.2 ConfigMap을 볼륨으로 마운트하기

ConfigMap 데이터를 파일로 마운트할 수 있습니다. 이 경우, ConfigMap의 각 키가 파일 이름이 되며, 그 값은 파일 내용으로 저장됩니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
    - name: myapp-container
      image: nginx:1.25.3-alpine
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

이 예시에서는 `app-config` ConfigMap의 데이터를 `/etc/config` 디렉토리에 마운트하고 있습니다.

### 2.3 ConfigMap 수정 및 적용

실행 중인 애플리케이션에서 ConfigMap을 수정할 수 있습니다. 예를 들어, 다음과 같이 ConfigMap을 수정하고 이를 적용할 수 있습니다.

```bash
kubectl edit configmap app-config
```

수정 후 Pod는 변경된 ConfigMap을 자동으로 사용할 수 있습니다. 필요하다면 Pod를 재시작하여 변경된 환경 변수를 반영할 수 있습니다.

## 3. Secret 활용

### 3.1 환경 변수를 통한 Secret 사용

Secret은 ConfigMap과 유사하게 Pod의 환경 변수로 사용할 수 있습니다. 아래는 Secret을 환경 변수로 사용하는 예시입니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
    - name: myapp-container
      image: busybox
      envFrom:
        - secretRef:
            name: db-secret
```

이 Pod는 `db-secret` Secret의 값을 환경 변수로 설정하여 컨테이너 내에서 사용할 수 있습니다.

### 3.2 Secret을 볼륨으로 마운트하기

Secret 데이터를 볼륨으로 마운트할 수 있습니다. 마운트된 Secret의 각 키가 파일 이름이 되고, 그 값은 파일 내용으로 저장됩니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
    - name: myapp-container
      image: busybox
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
  volumes:
    - name: secret-volume
      secret:
        secretName: db-secret
```

이 예시에서는 `db-secret` Secret을 `/etc/secrets` 디렉토리에 마운트하여 Pod에서 파일로 접근할 수 있습니다.

## 4. 실습: ConfigMap과 Secret 생성 및 활용

### 4.1 ConfigMap 생성

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=DATABASE_URL=postgres://user:password@db:5432/mydatabase
```

### 4.2 Secret 생성

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

### 4.3 Pod에서 ConfigMap과 Secret 사용

ConfigMap과 Secret을 생성한 후, 이를 Pod에서 사용하는 예시를 작성할 수 있습니다. 다음과 같이 ConfigMap과 Secret을 환경 변수로 설정한 Pod YAML 파일을 작성합니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-secret-pod
spec:
  containers:
    - name: myapp-container
      image: nginx:1.25.3-alpine
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: db-secre
```

### 4.4 ConfigMap과 Secret을 동적으로 변경

Kubernetes에서는 ConfigMap과 Secret의 변경 사항을 Pod에 즉시 반영할 수 있습니다. 변경 후, 다음 명령어를 사용하여 Pod를 재시작하지 않고도 업데이트된 값을 반영할 수 있습니다.

```bash
kubectl rollout restart pod <pod-name>
```

## 5. Ambassador Pod 패턴

### Ambassador 패턴 이해

Ambassador 패턴은 외부 서비스에 대한 요청을 처리하는 별도의 컨테이너(Proxy)를 Pod에 추가하여 외부 서비스와의 연결을 관리합니다. 이 패턴은 주로 데이터베이스와 같은 외부 리소스에 대한 접근을 제어하고, 복잡한 네트워크 연결을 간소화하는 데 사용됩니다.

```yaml
yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-pod
spec:
  containers:
    - name: app-container
      image: nginx:1.25.3-alpine
    - name: ambassador-container
      image: nginx:1.25.3-alpine
```

Ambassador 컨테이너는 외부 DB 서버나 API와의 연결을 관리하며, 애플리케이션 컨테이너는 이 Ambassador 컨테이너를 통해 외부 서비스에 접근합니다.