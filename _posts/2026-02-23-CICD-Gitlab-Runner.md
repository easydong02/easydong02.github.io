---
title: "[CI/CD] GitLab Runner 완전 정리: Kubernetes Executor 설정과 동작 원리"
date: 2026-02-23 00:00:00 +0900
categories: [Infra, CICD]
tags: [gitlab, gitlab-runner, kubernetes, cicd, executor, pipeline, docker, tekton]
render_with_liquid: false
---

# GitLab Runner

## 1. 개요

**한 줄 요약**: GitLab CI/CD 파이프라인을 실행하는 에이전트

**언제 사용하나**:
- GitLab `.gitlab-ci.yml`에 정의된 CI/CD Job을 실행할 때
- 코드 푸시/MR 이벤트 발생 시 자동으로 빌드/테스트 수행
- Kubernetes 클러스터에서 동적으로 Pod를 생성하여 Job 실행

**동작 흐름**:

```
GitLab 서버 → Runner 등록 → Job 할당 → Executor에서 실행 → 결과 반환
```

---

## 2. 핵심 개념

### 2.1. Executor 종류

| Executor | 설명 | 사용 시점 |
|----------|------|-----------|
| **Kubernetes** | Kubernetes 클러스터에서 Job마다 Pod 생성 | 가장 일반적, 동적 스케일링 |
| **Docker** | Docker 컨테이너에서 실행 | 단일 서버 환경 |
| **Shell** | Runner가 설치된 서버에서 직접 실행 | 간단한 환경, 레거시 |

### 2.2. Runner 타입

| 타입 | 범위 | 설명 |
|------|------|------|
| **Shared Runner** | GitLab 인스턴스 전체 | 모든 프로젝트에서 사용 가능 |
| **Group Runner** | 특정 그룹 | 해당 그룹의 프로젝트에서만 사용 |
| **Specific Runner** | 특정 프로젝트 | 지정된 프로젝트에만 할당 |

### 2.3. 주요 특징

- **동적 스케일링**: Kubernetes Executor는 Job마다 Pod 생성 후 종료
- **태그 기반 할당**: Runner에 태그를 지정해 특정 Job만 실행 가능
- **동시 실행 제어**: `concurrent` 설정으로 동시 실행 Job 수 제한

---

## 3. 기본 명령어

```bash
# Runner Pod 확인 (gitlab-runner namespace)
kubectl get pods -n gitlab-runner

# Runner 로그 확인
kubectl logs -n gitlab-runner -f

# Runner 설정 확인
kubectl get configmap -n gitlab-runner gitlab-runner -o yaml

# Runner가 실행 중인 Job Pod 확인
kubectl get pods -n gitlab-runner -l "app=gitlab-runner"

# 특정 Job Pod 로그 확인
kubectl logs -n gitlab-runner runner---
```

출력 예시:

```
NAME                                READY   STATUS    RESTARTS   AGE
gitlab-runner-5d8f7c9b8d-k7m2p      1/1     Running   0          5d

# Job 실행 중
runner-123-456-abc123               1/1     Running   0          30s
```

---

## 4. 주요 설정 패턴

### 4.1. Kubernetes Executor Runner ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gitlab-runner
  namespace: gitlab-runner
data:
  config.toml: |
    concurrent = 10       # 동시 실행 가능한 Job 수
    check_interval = 3

    [[runners]]
      name = "k8s-runner"
      url = "https://gitlab.example.com/"
      token = "RUNNER_TOKEN"
      executor = "kubernetes"

      [runners.kubernetes]
        namespace = "gitlab-runner"
        image = "alpine:latest"     # 기본 이미지
        privileged = false
        cpu_request = "100m"
        memory_request = "128Mi"
        cpu_limit = "1"
        memory_limit = "1Gi"
        service_cpu_request = "100m"
        service_memory_request = "128Mi"
        helper_cpu_request = "50m"
        helper_memory_request = "64Mi"

        # PVC로 빌드 캐시 저장
        [runners.kubernetes.volumes.pvc]
          name = "gitlab-runner-cache"
          mount_path = "/cache"
```

**특징**:
- Job마다 `alpine:latest` 기반 Pod 생성
- `.gitlab-ci.yml`에서 `image:` 지정 시 해당 이미지로 덮어씀
- 리소스 제한으로 클러스터 과부하 방지

### 4.2. .gitlab-ci.yml 예시

```yaml
stages:
  - build
  - test

variables:
  MAVEN_OPTS: "-Dmaven.repo.local=/cache/.m2"

build-job:
  stage: build
  image: maven:3.8-openjdk-11    # Runner의 기본 이미지 덮어씀
  tags:
    - kubernetes                 # 이 태그를 가진 Runner에서만 실행
  script:
    - mvn clean package -DskipTests
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 hour
  only:
    - main
    - merge_requests

test-job:
  stage: test
  image: maven:3.8-openjdk-11
  tags:
    - kubernetes
  script:
    - mvn test
  dependencies:
    - build-job
```

**동작 흐름**:

```
1. GitLab 서버가 `kubernetes` 태그를 가진 Runner에 Job 할당
   ↓
2. Runner가 Kubernetes에 `maven:3.8-openjdk-11` 이미지로 Pod 생성
   ↓
3. Pod 내부에서 `mvn clean package` 실행
   ↓
4. 빌드 산출물을 artifacts로 저장
   ↓
5. Pod 종료 후 자동 삭제
```

---

## 5. GitLab Runner vs Tekton 비교

| 항목 | GitLab Runner | Tekton |
|------|---------------|--------|
| **트리거** | GitLab 이벤트 (push, MR) | 수동 또는 EventListener |
| **설정 위치** | `.gitlab-ci.yml` (소스 저장소) | Pipeline YAML (클러스터) |
| **의존성** | GitLab 서버 필요 | 독립적 (K8s만 필요) |
| **재사용성** | Job 템플릿 (include) | Task 재사용 (taskRef) |
| **UI** | GitLab CI/CD UI | Tekton Dashboard |
| **캐싱** | 기본 지원 (artifacts, cache) | 별도 구성 필요 |
| **병렬 실행** | `parallel` 키워드 | DAG 방식 |

---

## 6. 주의사항

### 버전 호환성

- K8s 1.21: GitLab Runner 14.x 이상 권장
- GitLab 15.3.2-ce: Runner 15.x와 호환성 좋음
- Runner와 GitLab 버전 차이가 **2 major 버전 이상**이면 호환성 문제 발생 가능

### HyperCloud IC에서 GitLab Runner로 전환하는 이유

1. **표준화**: GitLab CI/CD는 업계 표준, HyperCloud IC는 Tmax 종속
2. **HyperCloud 독립성**: EKS 마이그레이션 등 플랫폼 전환 대비
3. **성능**: GitLab Runner가 캐싱/병렬 실행에서 더 우수
4. **생태계**: GitLab CI/CD 커뮤니티 및 레퍼런스가 풍부