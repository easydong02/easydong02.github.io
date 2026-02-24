---
title: "[CI/CD] Tekton vs HyperCloud Integration Config 완전 정리"
date: 2026-02-24 00:00:00 +0900
categories: [Infra, CICD]
tags: [tekton, hypercloud, integration-config, cicd, kubernetes, pipeline, operator, gitlab]
render_with_liquid: false
---

# Tekton vs HyperCloud Integration Config

**핵심 결론**: Integration Config(IC)는 **Tekton을 감싼 추상화 레이어**입니다. IC는 껍데기이고, **Tekton이 실제로 일하는 엔진**입니다.

---

## 1. 전체 구조

```
┌─────────────────────────────────────────────────────────────┐
│ HyperCloud Console (UI)                                     │
│  - 사용자가 파이프라인 설정하는 웹 인터페이스                          │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Integration Config (ConfigMap)                              │
│  - 파이프라인 템플릿 (YAML)                                      │
│  - HyperCloud만의 추상화 문법                                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ CICD Operator (0.4.14)                                      │
│  - IC를 읽어서 Tekton 리소스로 변환                               │
│  - Kubernetes Custom Controller                             │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Tekton Pipelines (0.26.0)                                   │
│  - 실제 파이프라인 실행 엔진 (오픈소스)                             │
│  - Task, Pipeline, PipelineRun 관리                          │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│ Kubernetes Pods                                             │
│  - compile, build, scan, push, sign 각 단계 실행               │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 각 컴포넌트의 역할

### 2.1. Tekton Pipelines — 실행 엔진

Kubernetes-native CI/CD 프레임워크 (오픈소스)

**핵심 리소스**:

```yaml
# Task: 재사용 가능한 작업 단위
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: maven-compile
spec:
  steps:
  - name: compile
    image: scr-maven-img-aicd
    script: |
      mvn clean package
```

```yaml
# Pipeline: Task들의 순서 정의
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-pipeline
spec:
  tasks:
  - name: compile
    taskRef:
      name: maven-compile
  - name: build-image
    taskRef:
      name: buildah-build
    runAfter:
    - compile   # compile 완료 후 실행
```

```yaml
# PipelineRun: Pipeline 실행 인스턴스
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-pipeline-run-20250122-001
spec:
  pipelineRef:
    name: build-pipeline
  params:
  - name: git-url
    value: https://gitlab.example.com/myapp.git
```

**Tekton의 역할**:
- Pod 생성 및 실행
- Task 간 데이터 전달 (Workspace, Results)
- 실행 상태 추적
- 로그 수집

---

### 2.2. Integration Config (IC) — 추상화 레이어

HyperCloud가 만든 Tekton 위의 설정 간소화 도구

**IC 예시**:

```yaml
apiVersion: cicd.tmax.io/v1
kind: IntegrationConfig
metadata:
  name: myapp-pipeline
  namespace: shg-cicd-aicd
spec:
  git:
    type: gitlab
    url: https://gitlab.example.com/myapp.git
    token: gitlab-token-secret

  jobs:
    preSubmit:   # PR/MR 시 실행
    - name: test
      image: maven:3.8
      script: mvn test

    postSubmit:  # merge 후 실행
    - name: compile
      image: scr-maven-img-aicd
      script: /usr/local/s2i/bin/assemble

    - name: build-image
      image: buildah:v1.23.3
      script: buildah bud -t myapp:latest
      when:
      - key: refs
        value: refs/heads/main

    - name: push-image
      image: buildah:v1.23.3
      script: buildah push myapp:latest

  triggers:
  - type: gitlab
    gitlab:
      push:
        branch: main
```

**IC의 역할**:
- 간소화된 문법 (Git 연동, 트리거 자동 설정)
- HyperCloud Console UI와 연동
- Webhook 자동 생성
- 직접 실행 안 함 → Tekton으로 변환만

---

### 2.3. CICD Operator — 번역기

IC를 Tekton 리소스로 변환하는 Kubernetes Operator

**동작 흐름**:

```
1. 사용자가 IC 생성
   ↓
2. CICD Operator가 감지 (Watch)
   ↓
3. IC 파싱 및 검증
   ↓
4. Tekton 리소스 자동 생성:
   - Task (compile, build, push)
   - Pipeline (전체 흐름)
   - TriggerTemplate (Git Webhook용)
   - TriggerBinding (파라미터 매핑)
   - EventListener (Webhook 수신)
   ↓
5. GitLab Webhook 자동 등록
```

**Operator가 생성하는 Tekton 리소스 확인**:

```bash
# IC 하나당 생성되는 리소스들
kubectl get task,pipeline,triggertemplate -n shg-cicd-aicd

# 출력 예시:
# task.tekton.dev/myapp-compile
# task.tekton.dev/myapp-build
# task.tekton.dev/myapp-push
# pipeline.tekton.dev/myapp-pipeline
# triggertemplate.tekton.dev/myapp-trigger
# eventlistener.tekton.dev/myapp-listener
```

---

## 3. 실제 실행 흐름 (코드 푸시 시나리오)

```
1. 개발자 git push (GitLab)
   ↓
2. GitLab Webhook 호출
   POST http://eventlistener-svc/myapp-listener
   ↓
3. EventListener (Tekton Trigger)
   - Webhook 수신
   - TriggerBinding으로 파라미터 추출
   ↓
4. TriggerTemplate
   - PipelineRun 생성
   ↓
5. Tekton Pipeline Controller
   - Pipeline 정의 읽기
   - Task 순서대로 실행
   ↓
6. Task 실행 (각각 별도 Pod)
   ├─ compile Pod 생성 (scr-maven-img-aicd)
   │  └─ mvn package 실행
   ├─ build Pod 생성 (buildah)
   │  └─ buildah bud 실행
   └─ push Pod 생성 (buildah)
      └─ buildah push 실행
   ↓
7. 각 Pod 완료 후 자동 삭제
   ↓
8. PipelineRun 상태 업데이트 (Succeeded/Failed)
   ↓
9. HyperCloud Console에서 결과 확인
```

---

## 4. IC vs Tekton 직접 사용 비교

### IC 방식 (HyperCloud)

```yaml
# ConfigMap으로 간단하게 정의
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-ic
data:
  template: |
    jobs:
      postSubmit:
      - name: build
        image: buildah
        script: buildah bud -t myapp
```

| 장점 | 단점 |
|------|------|
| 간단한 문법 | HyperCloud 전용 (이식성 낮음) |
| UI에서 관리 가능 | 고급 기능 제한 (복잡한 분기 등) |
| Git Webhook 자동 설정 | - |
| Tekton 몰라도 사용 가능 | - |

### Tekton 직접 사용 (표준 방식)

```yaml
# Task 정의
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: build-task
spec:
  steps:
  - name: build
    image: buildah
    script: buildah bud -t myapp
---
# Pipeline 정의
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-pipeline
spec:
  tasks:
  - name: build
    taskRef:
      name: build-task
---
# Trigger 정의 (Webhook)
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: gitlab-listener
spec:
  triggers:
  - name: gitlab-push
    bindings:
    - ref: gitlab-binding
    template:
      ref: build-template
```

| 장점 | 단점 |
|------|------|
| 완전한 제어 가능 | 학습 곡선 높음 |
| 이식성 (어디서나 사용 가능) | YAML이 많아짐 |
| 커뮤니티 Task 재사용 | Webhook 수동 설정 |

---

## 5. 비유로 이해하기

```
┌────────────────────────────────────────────┐
│ Integration Config                         │
│  = "한국어 레시피" (간단한 설명서)                │
│                                            │
│  예: "고기 굽고, 야채 볶고, 밥에 비벼라"           │
└────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│ CICD Operator                              │
│  = "번역기"                                  │
│    (레시피를 전문 요리사용 매뉴얼로 변환)           │
│                                            │
│  예: "1. 팬을 180도로 예열                     │
│       2. 고기를 3분간 앞뒤로 굽기                │
│       3. 야채는 센 불에 2분..."                │
└────────────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│ Tekton Pipelines                           │
│  = "주방 장비" (실제 요리 도구)                  │
│                                            │
│  예: 가스레인지(Pod), 냄비(Container)           │
└────────────────────────────────────────────┘
```

| 컴포넌트 | 역할 | 비유 |
|----------|------|------|
| **Integration Config** | 파이프라인 템플릿 (간소화 문법) | 한국어 레시피 |
| **CICD Operator** | IC → Tekton 변환 | 번역기 |
| **Tekton Pipelines** | 실제 CI/CD 실행 엔진 | 주방 장비 |
| **Kubernetes** | 인프라 (Pod 실행) | 주방 공간 |

---

## 6. 확인 명령어

### Tekton 리소스 확인

```bash
# IC로 생성된 Tekton 리소스들
kubectl get tasks,pipelines,pipelineruns -n shg-cicd-aicd

# 특정 IC에서 생성된 리소스 찾기
kubectl get task -n shg-cicd-aicd -l cicd.tmax.io/integration-config=myapp-ic
```

### IC → Tekton 변환 확인

```bash
# IC 내용 확인
kubectl get ic myapp-ic -n shg-cicd-aicd -o yaml

# 생성된 Pipeline 확인
kubectl get pipeline -n shg-cicd-aicd -o yaml | grep -A 20 "name: myapp"

# Task 내용 확인
kubectl get task myapp-compile -n shg-cicd-aicd -o yaml
```

### 실행 중인 파이프라인 확인

```bash
# PipelineRun 목록
tkn pipelinerun list -n shg-cicd-aicd

# 실시간 로그
tkn pipelinerun logs -f <pipelinerun-name> -n shg-cicd-aicd

# 각 Task의 Pod 확인
kubectl get pods -n shg-cicd-aicd -l tekton.dev/pipelineRun=<pipelinerun-name>
```

### CICD Operator 로그 확인

```bash
# Operator가 IC를 처리하는 과정 확인
kubectl logs -n cicd-system -l app=cicd-operator -f

# IC 변환 이벤트 확인
kubectl get events -n shg-cicd-aicd | grep IntegrationConfig
```

---

## 7. HyperCloud의 전략

### IC를 만든 이유

- **진입 장벽 낮추기**: Tekton은 Kubernetes 전문가용이지만, IC는 개발자도 쉽게 사용 가능
- **HyperCloud 생태계 통합**: HyperAuth(인증), HyperRegistry(이미지 저장소), HyperCloud Console(UI) 모두 IC와 자동 연동
- **관리 편의성**: ConfigMap 하나로 전체 파이프라인 관리, UI에서 수정 가능

### GitLab Runner로 전환하는 이유

- **표준화**: GitLab CI/CD는 업계 표준, HyperCloud IC는 Tmax 종속
- **HyperCloud 독립성**: EKS 마이그레이션 등 플랫폼 전환 대비
- **성능**: GitLab Runner가 캐싱/병렬 실행에서 더 우수
- **커뮤니티**: GitLab CI/CD 생태계 및 레퍼런스가 풍부

---

## 참고 자료

- HyperCloud CICD 공식 문서: <https://docs.hypercloud.com/cicd/>