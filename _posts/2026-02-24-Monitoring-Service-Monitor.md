---
title: "[Monitoring] ServiceMonitor 완전 정리: Prometheus Operator 메트릭 자동 수집"
date: 2026-02-24 00:00:00 +0900
categories: [Infra, Monitoring]
tags: [prometheus, servicemonitor, prometheus-operator, kubernetes, monitoring, grafana, relabeling, rbac]
render_with_liquid: false
---

# ServiceMonitor

## 1. 개요

**한 줄 요약**: Prometheus Operator가 모니터링 대상을 자동으로 발견하고 메트릭을 수집하기 위한 CRD (Custom Resource Definition)

**언제 사용하나**:
- Prometheus에서 특정 Service의 메트릭을 자동으로 수집하고 싶을 때
- 애플리케이션 배포 시 모니터링 설정을 자동화할 때
- Prometheus 설정 파일을 직접 수정하지 않고 선언적으로 타겟을 관리할 때

**동작 흐름**:

```
ServiceMonitor 생성
  → Prometheus Operator 감지
  → Prometheus 설정 자동 업데이트 (Hot Reload)
  → Service Endpoint 스크래핑
```

**구성 요소 관계**:

```
ServiceMonitor (CRD)
  → Service (Label Selector로 탐색)
  → Endpoint (Pod)
  → Metrics Endpoint
```

---

## 2. 핵심 개념

### 2.1. Prometheus Operator의 역할

- ServiceMonitor, PodMonitor, PrometheusRule 등 CRD 관리
- ServiceMonitor를 Watch하여 Prometheus 설정 파일 자동 생성/업데이트
- Prometheus Pod 재시작 없이 Hot Reload

### 2.2. ServiceMonitor vs PodMonitor

| 항목 | ServiceMonitor | PodMonitor |
|------|----------------|------------|
| **타겟** | Service → Endpoint | Pod 직접 |
| **사용 시기** | Service가 있는 경우 (일반적) | Service 없이 Pod만 모니터링 |
| **장점** | Service 기반 로드밸런싱 | Pod 직접 접근, 더 유연 |
| **예시** | 웹 애플리케이션, API 서버 | StatefulSet, DaemonSet |

### 2.3. 주요 특징

- **자동 서비스 디스커버리**: Label Selector로 Service 자동 탐색
- **선언적 설정**: YAML로 모니터링 대상 정의
- **네임스페이스 격리**: 네임스페이스별로 ServiceMonitor 관리 가능
- **동적 업데이트**: ServiceMonitor 변경 시 자동 반영

---

## 3. 기본 명령어

```bash
# ServiceMonitor 목록 조회
kubectl get servicemonitor -n <namespace>

# ServiceMonitor 상세 확인
kubectl describe servicemonitor <name> -n <namespace>

# ServiceMonitor YAML 확인
kubectl get servicemonitor <name> -n <namespace> -o yaml

# Prometheus Operator가 ServiceMonitor를 인식했는지 확인
kubectl logs -n monitoring prometheus-operator-<pod-name> | grep -i servicemonitor

# ServiceMonitor가 참조하는 Service 확인
kubectl get svc -n <namespace> -l <label-selector>

# Prometheus UI에서 타겟 확인
# http://<prometheus-url>/targets
```

출력 예시:

```
NAME                      AGE
kube-state-metrics        5d
node-exporter             5d
my-app-monitor            2h
```

---

## 4. 실무 패턴

### 4.1. 기본 ServiceMonitor 구조

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  namespace: default
  labels:
    app: my-app
    prometheus: kube-prometheus  # Prometheus가 선택할 Label
spec:
  # 모니터링할 Service 선택 (Label 기반)
  selector:
    matchLabels:
      app: my-app

  # 모니터링할 네임스페이스 (생략 시 자신의 네임스페이스만)
  namespaceSelector:
    matchNames:
    - default

  # 메트릭 수집 엔드포인트 정의
  endpoints:
  - port: metrics        # Service의 포트 이름
    interval: 30s        # 스크래핑 주기
    path: /metrics       # 메트릭 경로
    scheme: http         # http 또는 https
```

**대응되는 Service**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: default
  labels:
    app: my-app         # ServiceMonitor의 selector와 반드시 일치
spec:
  selector:
    app: my-app
  ports:
  - name: metrics        # ServiceMonitor의 endpoints.port와 반드시 일치
    port: 8080
    targetPort: 8080
  - name: http
    port: 80
    targetPort: 8080
```

### 4.2. 여러 엔드포인트 모니터링

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: multi-port-monitor
  namespace: default
spec:
  selector:
    matchLabels:
      app: complex-app
  endpoints:
  - port: app-metrics      # 애플리케이션 메트릭
    interval: 30s
    path: /metrics
  - port: jvm-metrics      # JVM 메트릭
    interval: 60s
    path: /actuator/prometheus
  - port: custom-metrics   # 커스텀 메트릭
    interval: 15s
    path: /custom/metrics
```

**대응되는 Service**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: complex-app-service
  namespace: default
  labels:
    app: complex-app
spec:
  selector:
    app: complex-app
  ports:
  - name: app-metrics
    port: 9090
    targetPort: 9090
  - name: jvm-metrics
    port: 9091
    targetPort: 9091
  - name: custom-metrics
    port: 9092
    targetPort: 9092
```

### 4.3. 여러 네임스페이스 모니터링

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: cross-namespace-monitor
  namespace: monitoring
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      monitoring: "true"  # 이 Label을 가진 모든 Service 모니터링

  # 여러 네임스페이스 지정
  namespaceSelector:
    matchNames:
    - default
    - production
    - staging

  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

**각 네임스페이스의 Service에 Label 추가**:

```yaml
metadata:
  labels:
    monitoring: "true"  # 이 Label이 있으면 자동으로 수집 대상에 포함
```

### 4.4. Relabeling 및 메트릭 필터링

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: advanced-monitor
  namespace: default
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

    # Relabeling: 스크래핑 전 Label 추가/수정/삭제
    relabelings:
    # Kubernetes Label을 Prometheus Label로 추가
    - sourceLabels: [__meta_kubernetes_pod_name]
      targetLabel: pod_name

    # 특정 환경만 수집 (production만 keep)
    - sourceLabels: [__meta_kubernetes_pod_label_environment]
      regex: production
      action: keep

    # Label 이름 변경
    - sourceLabels: [__meta_kubernetes_namespace]
      targetLabel: namespace

    # Metric Relabeling: 수집 후 처리
    metricRelabelings:
    # 특정 메트릭만 유지
    - sourceLabels: [__name__]
      regex: "(http_requests_total|http_request_duration_seconds.*)"
      action: keep

    # 불필요한 Label 제거
    - regex: "pod_template_hash"
      action: labeldrop
```

### 4.5. HTTPS 및 인증 엔드포인트

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: secure-monitor
  namespace: default
spec:
  selector:
    matchLabels:
      app: secure-app
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
    scheme: https

    # TLS 설정
    tlsConfig:
      ca:
        secret:
          name: prometheus-ca
          key: ca.crt
      insecureSkipVerify: false  # 운영 환경은 false 유지

    # Basic Auth
    basicAuth:
      username:
        name: prometheus-auth
        key: username
      password:
        name: prometheus-auth
        key: password
```

```bash
# 필요한 Secret 생성
kubectl create secret generic prometheus-auth \
  --from-literal=username=prometheus \
  --from-literal=password=secret123 \
  -n default

kubectl create secret generic prometheus-ca \
  --from-file=ca.crt=/path/to/ca.crt \
  -n default
```

### 4.6. Spring Boot Actuator 연동 예시

```yaml
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
  namespace: default
  labels:
    app: spring-app
    monitoring: enabled
spec:
  selector:
    app: spring-app
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  - name: actuator       # Actuator 전용 포트
    port: 8081
    targetPort: 8081
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: spring-app-monitor
  namespace: default
  labels:
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      app: spring-app
  endpoints:
  - port: actuator
    interval: 15s
    path: /actuator/prometheus   # Spring Boot Actuator 경로
    relabelings:
    - sourceLabels: [__meta_kubernetes_service_label_app]
      targetLabel: application
    - sourceLabels: [__meta_kubernetes_namespace]
      targetLabel: kubernetes_namespace
```

**Spring Boot `application.yml`**:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus,health,info
  metrics:
    export:
      prometheus:
        enabled: true
  server:
    port: 8081    # Actuator 전용 포트
```

---

## 5. 트러블슈팅

### 5.1. 흔한 실수 7가지

**1. ServiceMonitor와 Service의 Label 불일치**

```
❌ ServiceMonitor selector: app=my-app
   Service labels:          application=my-app
→ Prometheus가 Service를 발견하지 못함

✅ Label 정확히 일치시키기
   kubectl get svc -n <namespace> --show-labels  # Label 확인 후 맞추기
```

**2. Service 포트 이름 불일치**

```
❌ ServiceMonitor endpoints.port: metrics
   Service ports.name:           http
→ Prometheus가 포트를 찾지 못함

✅ 포트 이름 일치 또는 포트 번호로 직접 지정
   endpoints:
   - port: 8080    # 이름 대신 번호로 지정
```

**3. Prometheus가 ServiceMonitor를 선택하지 못함**

```
❌ ServiceMonitor labels:            app=my-app
   Prometheus serviceMonitorSelector: team=backend
→ Prometheus가 ServiceMonitor를 무시

✅ Prometheus CRD에서 serviceMonitorSelector 확인
   kubectl get prometheus -n monitoring -o yaml
```

**4. 네임스페이스 접근 권한 없음**

```
❌ 다른 네임스페이스 Service를 모니터링하려 했지만
   Prometheus ServiceAccount에 해당 네임스페이스 RBAC 권한 없음
→ Service를 발견하지 못함

✅ Prometheus ServiceAccount에 권한 부여
```

**5. 메트릭 엔드포인트 경로 오류**

```
❌ endpoints.path: /metrics
   실제 애플리케이션:  /actuator/prometheus
→ HTTP 404 에러, 메트릭 수집 실패

✅ 실제 경로 확인 후 설정
   kubectl exec -it <pod-name> -n <namespace> -- curl localhost:8080/actuator/prometheus
```

**6. interval이 너무 짧음**

```
❌ interval: 1s
→ Prometheus 과부하, 스토리지 급증

✅ 15s ~ 60s 권장 (애플리케이션 중요도에 따라 조정)
```

**7. prometheus-operator 로그 미확인**

```
❌ ServiceMonitor 생성했는데 타겟에 안 보임
→ Operator에서 에러 발생 중임에도 로그 미확인

✅ Operator 로그 최우선 확인
   kubectl logs -n monitoring prometheus-operator-<pod> | grep -i error
```

### 5.2. prometheus-operator 트러블슈팅 명령어

```bash
# 1. prometheus-operator Pod 상태 확인
kubectl get pods -n monitoring | grep prometheus-operator

# 2. ServiceMonitor 관련 에러 확인
kubectl logs -n monitoring prometheus-operator-<pod-name> | grep -i servicemonitor
kubectl logs -n monitoring prometheus-operator-<pod-name> | grep -i error

# 흔한 에러 메시지:
# - "unable to sync ServiceMonitor"       → RBAC 권한 문제
# - "selector does not match any services" → Label 불일치
# - "endpoints not found"                 → Service에 해당 포트 없음

# 3. Prometheus 내부 설정에 반영되었는지 확인
kubectl exec -it -n monitoring prometheus-k8s-0 -- sh
cat /etc/prometheus/config_out/prometheus.env.yaml | grep -A 10 my-app

# 4. Prometheus의 serviceMonitorSelector 확인
kubectl get prometheus -n monitoring kube-prometheus -o yaml | grep -A 5 serviceMonitorSelector

# 5. Prometheus UI에서 타겟 확인 (포트 포워딩)
kubectl port-forward -n monitoring svc/prometheus-k8s 9090:9090
# 브라우저: http://localhost:9090/targets

# 6. ServiceMonitor YAML 사전 검증
kubectl apply --dry-run=client -f servicemonitor.yaml

# 7. ServiceMonitor 처리 이벤트 확인
kubectl get events -n monitoring | grep ServiceMonitor

# 8. Service Endpoint 생성 여부 확인 (없으면 Pod Selector 문제)
kubectl get endpoints <service-name> -n <namespace>
```

### 5.3. ServiceMonitor 생성 후 확인 순서

```bash
# Step 1: ServiceMonitor 생성 확인
kubectl get servicemonitor <name> -n <namespace>

# Step 2: prometheus-operator 로그 확인
kubectl logs -n monitoring prometheus-operator-<pod> | tail -50

# Step 3: Prometheus UI에서 타겟 확인
# http://<prometheus-url>/targets

# Step 4: 메트릭 수집 확인 (Prometheus Graph)
up{job="<service-name>"}
```

### 5.4. 타겟이 안 보일 때 빠른 디버깅

```bash
# Service Label 확인
kubectl get svc -n <namespace> --show-labels

# ServiceMonitor selector 확인
kubectl get servicemonitor <name> -n <namespace> -o yaml | grep -A 5 selector

# Service Endpoint 확인
kubectl get endpoints <service-name> -n <namespace>

# Pod 메트릭 엔드포인트 직접 확인
kubectl exec -it <pod-name> -n <namespace> -- curl localhost:8080/metrics
```

### 5.5. CrashLoopBackOff 대응

```bash
# 1. CrashLoopBackOff 상태 확인
kubectl get pods -n monitoring prometheus-operator-<pod>

# 2. 이전 로그 확인
kubectl logs -n monitoring prometheus-operator-<pod> --previous

# 3. ServiceMonitor 문법 오류 확인
kubectl apply --validate=true --dry-run=client -f servicemonitor.yaml

# 4. RBAC 권한 부족 확인
kubectl get clusterrole prometheus-operator -o yaml

# 5. Prometheus의 serviceMonitorSelector 확인
kubectl get prometheus -n monitoring -o jsonpath='{.items[0].spec.serviceMonitorSelector}'

# 6. 문제가 되는 ServiceMonitor 삭제 후 Operator 정상화 확인
kubectl delete servicemonitor <problematic-monitor> -n <namespace>
kubectl get pods -n monitoring prometheus-operator-<pod>
```

---

## 6. HyperCloud 환경 참고사항

- `monitoring` 네임스페이스에 Prometheus Operator 배포됨
- Prometheus Operator 버전: K8s 1.21 기준 0.50+ 권장
- ServiceMonitor API: `monitoring.coreos.com/v1`

**Grafana 연동 흐름**:

```
ServiceMonitor → Prometheus (메트릭 저장)
              → Grafana (Data Source: Prometheus 연결)
              → PromQL로 대시보드 구성
```

**금융권 환경 주의사항**:
- 메트릭에 민감한 정보 노출 금지 (계좌번호, 개인정보 등)
- 메트릭 엔드포인트는 내부망에서만 접근
- `relabeling`으로 불필요한 Label 제거

**성능 최적화**:
- `interval`: 애플리케이션 중요도에 따라 15s ~ 60s로 조정
- `metricRelabelings`으로 불필요한 메트릭 필터링
- Prometheus Retention 기간 적절히 설정 (기본 15일)

---

## 참고 자료

- 공식 문서: <https://prometheus-operator.dev/docs/operator/api/#monitoring.coreos.com/v1.ServiceMonitor>
- Prometheus Operator GitHub: <https://github.com/prometheus-operator/prometheus-operator>