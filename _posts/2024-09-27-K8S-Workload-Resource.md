---
title: "[Kubernetes] 워크로드 리소스(Workload Resource)"
date: 2024-09-27 00:00:00 +0900
categories: [Infra, Kubernetes]
tags: [kubernetes, daemonset, deployment, hpa, job, cronjob]
render_with_liquid: false
---

# Kubernetes Workload Resources: Pod를 위한 리소스 관리

Kubernetes에서 애플리케이션을 관리할 때 가장 중요한 요소 중 하나는 **Workload Resources**입니다. **Pod**는 Kubernetes의 기본 실행 단위이며, 이들 Pod를 효율적으로 관리하기 위해 다양한 **Workload Resources**가 필요합니다. 이번 글에서는 Kubernetes의 대표적인 Workload Resources인 **Deployment**, **StatefulSet**, **DaemonSet**, **Job**, **CronJob**, 그리고 **Horizontal Pod Autoscaler (HPA)**를 살펴보겠습니다.

## 1. Deployment

**Deployment**는 Kubernetes에서 가장 일반적으로 사용하는 리소스로, Pod의 배포 및 업데이트를 관리합니다. Deployment는 여러 Pod를 복제하여 애플리케이션을 확장하고, 배포 전략에 따라 롤링 업데이트 및 롤백을 처리할 수 있습니다.

### Deployment의 주요 기능

- **복제본 관리**: Pod의 복제본을 관리하여 일정 수의 Pod가 항상 실행되도록 보장.
- **롤링 업데이트**: 기존 Pod를 단계적으로 교체하여 애플리케이션을 업데이트.
- **롤백**: 문제가 발생한 경우 이전 상태로 되돌리기 가능.

### Deployment 예시

```yaml
yaml
코드 복사
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: nginx:1.21
        ports:
        - containerPort: 80
```

이 설정은 `nginx:1.21` 이미지를 사용하는 3개의 Pod 복제본을 생성하고 관리합니다.

## 2. StatefulSet

**StatefulSet**은 상태를 유지해야 하는 애플리케이션을 위한 리소스입니다. StatefulSet은 각 Pod에 고유한 네트워크 ID와 영구적인 스토리지를 제공하여 Pod가 재시작되어도 상태를 유지할 수 있게 합니다.

### StatefulSet의 주요 기능

- **고유한 네트워크 ID**: 각 Pod는 고유한 정적인 네트워크 ID를 가짐.
- **영구 스토리지**: Pod가 삭제되고 다시 생성되어도 데이터를 유지.
- **순차적인 배포**: Pod가 정해진 순서대로 배포되고 종료됨.

### StatefulSet 예시

```yaml
yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: example-statefulset
spec:
  serviceName: "example-service"
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: nginx:1.21
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

## 3. DaemonSet

**DaemonSet**은 클러스터의 각 노드에 하나의 Pod를 배포하는 리소스입니다. 주로 노드별 로그 수집기, 모니터링 에이전트 등을 배포할 때 사용됩니다.

### DaemonSet의 주요 기능

- **노드마다 하나의 Pod**: 클러스터 내 모든 노드에 동일한 Pod가 배포됨.
- **노드 추가 시 자동 배포**: 새로운 노드가 클러스터에 추가되면 DaemonSet은 해당 노드에 자동으로 Pod를 배포.

### DaemonSet 예시

```yaml
yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: example-daemonset
spec:
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: fluentd-elasticsearch:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
```

## 4. Job & CronJob

**Job**은 일회성 작업을 실행할 때 사용되며, 작업이 완료될 때까지 Pod를 실행합니다. **CronJob**은 일정한 시간에 반복적으로 Job을 실행할 때 사용됩니다.

### Job 예시

```yaml
yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: example-container
        image: busybox
        command: ["echo", "Hello, Kubernetes!"]
      restartPolicy: Never
```

### CronJob 예시

```yaml
yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: example-container
            image: busybox
            command: ["date"]
          restartPolicy: OnFailure
```

## 5. Horizontal Pod Autoscaler (HPA)

- *Horizontal Pod Autoscaler (HPA)**는 Pod의 리소스 사용량(CPU, 메모리 등)에 따라 자동으로 Pod의 수를 조정합니다. HPA는 동적으로 애플리케이션의 부하를 처리할 수 있도록 Pod의 수를 늘리거나 줄이는 데 사용됩니다.

### HPA 예시

```yaml
yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: example-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: example-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

HPA는 `example-deployment`의 CPU 사용량이 80%를 넘으면 Pod 수를 최대 10개까지 자동으로 확장합니다.