---
title: "[Kubernetes] Liveness & Readiness Probe 완전 정리: 헬스체크 메커니즘과 실무 패턴"
date: 2026-03-04 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, probe, liveness, readiness, startup, healthcheck, rolling-update, spring-boot]
render_with_liquid: false
---

# Liveness Probe & Readiness Probe

## 1. 개요

**한 줄 요약**: 컨테이너 상태를 주기적으로 검사하여 자동 복구 및 트래픽 제어를 수행하는 헬스체크 메커니즘

**언제 사용하나**:
- 애플리케이션이 응답하지 않을 때 Pod 자동 재시작 (Liveness)
- 애플리케이션 시작 완료 전까지 트래픽 차단 (Readiness)
- 로드밸런서에서 unhealthy Pod 자동 제외

**관련 기술**: Service, Deployment, Rolling Update, Ingress

---

## 2. 핵심 개념

### 2.1. Probe 3종류

**Liveness Probe (생존 확인)**

```
목적: 컨테이너가 살아있는지 확인
실패 시: kubelet이 컨테이너 재시작
사용 예: 데드락, 무한루프 등으로 응답 불가 상태 감지
```

**Readiness Probe (준비 상태 확인)**

```
목적: 컨테이너가 트래픽을 받을 준비가 되었는지 확인
실패 시: Service의 Endpoint에서 제외 (트래픽 차단)
사용 예: 초기화 작업 완료 대기, DB 연결 확인
```

**Startup Probe (시작 확인)** — K8s 1.16+

```
목적: 느린 시작 시간을 가진 애플리케이션 보호
실패 시: 컨테이너 재시작
사용 예: 레거시 앱, 대용량 데이터 로딩
```

### 2.2. 주요 특징

- **독립적 동작**: Liveness와 Readiness는 서로 독립적으로 판단
- **주기적 검사**: `periodSeconds`마다 반복 실행
- **실패 허용**: `failureThreshold` 횟수만큼 실패해야 액션 수행

### 2.3. 검사 방식 3가지

| 방식 | 설명 | 사용 시기 |
|------|------|-----------|
| **httpGet** | HTTP GET 요청으로 상태 확인 | REST API, 웹 애플리케이션 |
| **tcpSocket** | TCP 포트 연결 확인 | DB, Redis 등 포트만 확인 |
| **exec** | 컨테이너 내부 명령어 실행 | 커스텀 스크립트 필요 시 |

### 2.4. 주요 파라미터

| 파라미터 | 설명 | 기본값 |
|----------|------|--------|
| `initialDelaySeconds` | 첫 검사까지 대기 시간 | 0초 |
| `periodSeconds` | 검사 주기 | 10초 |
| `timeoutSeconds` | 응답 대기 시간 | 1초 |
| `failureThreshold` | 실패 허용 횟수 | 3회 |
| `successThreshold` | 성공 판정 횟수 | 1회 |

---

## 3. 기본 명령어

```bash
# Probe 설정 확인
kubectl describe pod <pod-name> -n <namespace>

# Probe 실패 이벤트 확인
kubectl get events -n <namespace> | grep -i "liveness\|readiness"

# Pod 상태 확인
kubectl get pod <pod-name> -n <namespace> \
  -o jsonpath='{.status.containerStatuses[0].state}'

# Probe 실패로 재시작된 횟수 확인
kubectl get pod <pod-name> -n <namespace> \
  -o jsonpath='{.status.containerStatuses[0].restartCount}'
```

**이벤트 출력 예시**:

```
Events:
  Warning  Unhealthy  2m  kubelet  Liveness probe failed: HTTP probe failed with statuscode: 500
  Warning  Unhealthy  1m  kubelet  Readiness probe failed: Get "http://10.0.1.5:8080/health": connection refused
```

---

## 4. 실무 패턴

### 4.1. HTTP 기반 Probe (가장 일반적)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.21
        ports:
        - containerPort: 8080

        livenessProbe:
          httpGet:
            path: /healthz          # 헬스체크 엔드포인트
            port: 8080
            httpHeaders:
            - name: Custom-Header
              value: Awesome
          initialDelaySeconds: 30   # 첫 검사까지 대기 (앱 시작 시간 고려)
          periodSeconds: 10         # 검사 주기
          timeoutSeconds: 5         # 응답 대기 시간
          failureThreshold: 3       # 3번 실패 시 재시작
          successThreshold: 1

        readinessProbe:
          httpGet:
            path: /ready            # 준비 상태 엔드포인트
            port: 8080
          initialDelaySeconds: 5    # Liveness보다 짧게 설정
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
```

**동작 시나리오**:

```
1. Pod 시작 후 5초:  Readiness Probe 시작 → 실패 시 Service에서 제외
2. Pod 시작 후 30초: Liveness Probe 시작  → 실패 시 재시작
3. /ready 성공:      Service Endpoint에 추가 → 트래픽 수신 시작
4. /healthz 3번 연속 실패: Pod 재시작
```

### 4.2. TCP Socket Probe (DB, Redis 등)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  template:
    spec:
      containers:
      - name: redis
        image: redis:6.2
        ports:
        - containerPort: 6379

        livenessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
```

HTTP 엔드포인트가 없는 경우 사용합니다. 포트 연결만 확인하므로 실제 서비스 로직은 확인하지 않습니다. Redis, MySQL, PostgreSQL 등에 적합합니다.

### 4.3. Exec Command Probe (커스텀 스크립트)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
spec:
  template:
    spec:
      containers:
      - name: postgres
        image: postgres:13

        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - pg_isready -U postgres
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - pg_isready -U postgres && psql -U postgres -c 'SELECT 1'
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 5
          failureThreshold: 3
```

**활용 예시**:
- PostgreSQL: `pg_isready`로 DB 연결 확인
- MySQL: `mysqladmin ping`
- 파일 존재 확인: `test -f /tmp/healthy`

exec 방식은 매번 쉘 프로세스를 생성하므로 httpGet보다 리소스 소모가 큽니다. 단, `&&` 연산자로 여러 조건을 한 번에 체크할 수 있다는 장점이 있습니다.

### 4.4. Startup Probe (느린 시작 애플리케이션)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legacy-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: legacy-app:1.0

        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 30      # 최대 300초(5분) 대기
          successThreshold: 1

        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

Startup Probe가 성공할 때까지 Liveness/Readiness는 실행되지 않습니다. 최대 `30회 × 10초 = 300초`까지 시작 시간을 허용하며, 성공 후 Liveness/Readiness가 활성화됩니다.

### 4.5. Spring Boot 실무 권장 설정

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: spring-app:latest
        ports:
        - containerPort: 8080

        livenessProbe:
          httpGet:
            path: /actuator/health/liveness   # Spring Boot Actuator
            port: 8080
          initialDelaySeconds: 60             # Spring Boot 시작 시간 고려
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3

        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
```

**Spring Boot `application.yml` 설정**:

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

---

## 5. 트러블슈팅

### 5.1. 디버깅 순서

```bash
# Step 1: Probe 실패 원인 확인 (Events 섹션)
kubectl describe pod <pod-name> -n <namespace>

# Step 2: 재시작 직전 로그 확인
kubectl logs <pod-name> -n <namespace> --previous

# Step 3: Probe 엔드포인트 직접 테스트
kubectl exec -it <pod-name> -n <namespace> -- curl http://localhost:8080/healthz

# Step 4: Probe 설정 확인
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 10 livenessProbe

# Step 5: Service Endpoint 포함 여부 확인
kubectl get endpoints <service-name> -n <namespace>
# Readiness 실패 → Endpoint에서 제외됨
# Liveness 실패 → 재시작되지만 Endpoint는 유지
```

### 5.2. 흔한 실수 7가지

**1. Liveness와 Readiness를 동일하게 설정**

```
❌ 둘 다 동일한 엔드포인트와 설정 사용
→ 초기화 중 Liveness 실패로 무한 재시작

✅ Liveness는 치명적 오류만 감지
   Readiness는 일시적 오류도 감지
```

**2. initialDelaySeconds 너무 짧게 설정**

```
❌ initialDelaySeconds: 5 (Spring Boot는 30초 이상 소요)
→ 시작 전에 Liveness 실패로 재시작 반복

✅ 애플리케이션 시작 시간보다 길게 설정
   또는 Startup Probe 사용
```

**3. periodSeconds 너무 짧게 설정**

```
❌ periodSeconds: 1 (1초마다 검사)
→ 과도한 네트워크 요청, 애플리케이션 부하

✅ Liveness: 10초, Readiness: 5초 권장
```

**4. failureThreshold 너무 낮게 설정**

```
❌ failureThreshold: 1 (1번 실패로 재시작)
→ 일시적 네트워크 문제로 불필요한 재시작

✅ 3 이상 권장 (일시적 오류 허용)
```

**5. 무거운 헬스체크 로직**

```
❌ /healthz에서 DB 전체 테이블 스캔
→ Probe 요청마다 서비스 부하 증가

✅ 가벼운 로직만 수행 (SELECT 1 수준)
```

**6. Readiness Probe 미설정**

```
❌ Readiness Probe 없이 배포
→ Rolling Update 시 초기화 안 된 Pod로 트래픽 유입

✅ 반드시 Readiness Probe 설정
```

**7. Liveness와 Readiness에 동일 엔드포인트 혼용**

```
❌ 하나의 /health 엔드포인트를 양쪽에서 다른 의미로 사용
→ 혼란 및 예상치 못한 동작

✅ 엔드포인트 분리
   Liveness:  /healthz
   Readiness: /ready
```

---

## 6. 운영 참고사항

### 6.1. 권장 설정값

| Probe | initialDelaySeconds | periodSeconds | timeoutSeconds | failureThreshold |
|-------|--------------------:|--------------:|---------------:|-----------------:|
| **Liveness** | 30~60초 | 10초 | 5초 | 3회 |
| **Readiness** | 10~30초 | 5초 | 3초 | 3회 |
| **Startup** | 0초 | 10초 | 3초 | 30회 |

컨테이너 무게에 따른 `initialDelaySeconds` 기준:
- **경량** (Nginx, Redis): 짧게 설정 가능
- **중량** (Spring Boot, Java): 60초 이상 권장
- **DB 컨테이너**: Readiness에 실제 쿼리 실행 포함

### 6.2. Rolling Update 시나리오

```
1. 새 Pod 생성 → Readiness 성공 → Service Endpoint 추가
2. 트래픽 유입 → 구 Pod Readiness 실패 → Endpoint 제거
3. 구 Pod 종료 → 무중단 배포 완료
```

Readiness Probe 없이 Rolling Update를 수행하면 초기화가 완료되지 않은 새 Pod로 트래픽이 유입되어 오류가 발생합니다.

### 6.3. Prometheus 모니터링

```promql
# Pod 재시작 횟수 모니터링
kube_pod_container_status_restarts_total
```

**Alert 예시**:
- `KubePodCrashLooping`: 5분간 2회 이상 재시작 → Liveness Probe 설정 검토
- `KubePodNotReady`: 15분 이상 Ready 상태 아님 → Readiness Probe 설정 검토

### 6.4. 금융권 환경 주의사항

- 헬스체크 엔드포인트는 인증 없이 접근 가능하도록 설정 (Probe는 인증 헤더 불포함)
- 응답에 민감한 정보 노출 금지 (DB 연결 문자열, 버전 정보 등)
- Liveness: 애플리케이션 자체 생존 여부만 확인
- Readiness: 외부 의존성 (DB, Redis 등) 연결 상태 포함

### 6.5. CrashLoopBackOff 방지

- Startup Probe로 충분한 시작 시간 제공
- Liveness `initialDelaySeconds`를 넉넉하게 설정
- HyperCloud Console의 Pod 상태 화면에서 Probe 실패 이벤트 확인 가능

---

## 참고 자료

- Kubernetes 공식 문서: <https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/>
- Spring Boot Actuator: <https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html>