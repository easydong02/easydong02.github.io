---
title: "[Docker] Dockerfile 완전 정리: 이미지 빌드 원리와 멀티스테이지 빌드"
date: 2026-02-09 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, dockerfile, container, kubernetes, multi-stage-build, image, cicd]
render_with_liquid: false
---

# Dockerfile 완전 정리

## 1. 개요

Dockerfile은 Docker 이미지를 빌드하기 위한 설계도입니다. 컨테이너 내부 파일 시스템 구성, 실행 환경, 시작 명령어를 정의합니다.

### Docker vs Kubernetes 역할 분담

| 구분 | 주체 | 비유 | 역할 |
|------|------|------|------|
| **Build** | **Docker (Developer)** | **짐 싸는 사람** | 가방(이미지)을 열고 파일을 특정 경로(`/app`)에 넣고 잠금 |
| **Run** | **Kubernetes (Operator)** | **이사 업체** | 잠긴 가방을 목적지(노드)로 배송하고 지퍼를 열어줌(실행). 내용물은 건드리지 않음 |

> Kubernetes가 파일을 직접 넣는 경우는 **ConfigMap**, **Secret**, **PVC(Volume)** 마운트 때뿐입니다. 실행 파일(.jar 등)은 이미지 빌드 시 포함됩니다.

---

## 2. 파드 실행 시 이미지 다운로드 원리

### 이미지는 어디에 저장되는가?

Kubernetes Deployment를 실행하면 **마스터가 아니라 파드가 스케줄링된 워커 노드**가 직접 이미지를 다운로드합니다.

저장 위치는 컨테이너 런타임(CRI)에 따라 다릅니다:

| 런타임 | 저장 경로 |
|--------|-----------|
| **containerd** (최신 표준) | `/var/lib/containerd/` (레이어 단위 분산 저장) |
| **Docker** (구버전) | `/var/lib/docker/` |
| **CRI-O** | `/var/lib/containers/` |

### 동작 방식

```
1. 스케줄링: 마스터가 "Node A에서 파드를 실행해"라고 명령
   ↓
2. Pull: Node A의 Kubelet이 레지스트리에서 이미지 다운로드
   ↓
3. Cache: 이미지가 이미 Node A에 있으면 다운로드 생략 (캐시 사용)
```

---

## 3. 컨테이너 내부 파일의 출처

`kubectl exec`으로 파드에 접속했을 때 보이는 `/app/chatbot.jar` 같은 파일은 **Kubernetes가 넣은 것이 아닙니다.**

파일이 해당 경로에 있는 이유는 **Dockerfile에 그렇게 정의되어 있기 때문**입니다.

### Dockerfile 분석 예시

```dockerfile
# 1. 베이스 이미지 선언 (Java 환경)
FROM openjdk:17-jdk

# 2. 작업 디렉토리 설정
# 이 명령어 때문에 파일들이 /app 경로에 위치
WORKDIR /app

# 3. 파일 복사 (빌드 산출물을 이미지 내부로)
# 로컬의 build/libs/chatbot.jar → 이미지의 /app/chatbot.jar
COPY ./build/libs/chatbot.jar chatbot.jar

# 4. 컨테이너 시작 시 실행할 명령어
CMD ["java", "-jar", "chatbot.jar"]
```

---

## 4. Docker 이미지 레이어 구조

Docker 이미지는 **읽기 전용 레이어의 집합**입니다. 각 명령어(`FROM`, `RUN`, `COPY` 등)가 하나의 레이어를 생성합니다.

```
이미지 구조:
┌─────────────────┐
│ Layer 4: APP    │  ← 내 앱 (COPY)
├─────────────────┤
│ Layer 3: Maven  │  ← Maven 설치 (RUN)
├─────────────────┤
│ Layer 2: JDK    │  ← Java 설치
├─────────────────┤
│ Layer 1: OS     │  ← Ubuntu 등 (FROM)
└─────────────────┘
```

---

## 5. 단일 스테이지 빌드의 문제점

```dockerfile
FROM openjdk:11

# Maven 설치 (100MB)
RUN wget maven.tar.gz && tar -xzf maven.tar.gz
ENV MAVEN_HOME=/maven

# 소스 복사 및 빌드
COPY src /app/src
COPY pom.xml /app/
WORKDIR /app
RUN mvn clean package

CMD ["java", "-jar", "target/app.jar"]
```

**최종 이미지에 불필요하게 포함되는 것들:**

```
✅ app.jar (10MB)       ← 실제 필요한 것
❌ Maven (100MB)        ← 빌드 끝나면 불필요
❌ 의존성 jar들 (200MB) ← 이미 app.jar에 포함됨
❌ 소스코드 (50MB)      ← 빌드 끝나면 불필요

결과: 이미지 크기 500MB (실제 필요한 건 10MB)
```

---

## 6. 멀티스테이지 빌드

### 기본 원리

필요한 파일만 다음 스테이지로 복사하고, 빌드 환경 전체는 버립니다.

```dockerfile
# ===== Stage 1: 빌드 전용 =====
FROM maven:3.8.4-openjdk-11 AS builder
COPY src /app/src
COPY pom.xml /app/
WORKDIR /app
RUN mvn clean package
# 결과: /app/target/app.jar 생성

# ===== Stage 2: 실행 전용 =====
FROM openjdk:17-jdk-slim
COPY --from=builder /app/target/app.jar /app.jar
CMD ["java", "-jar", "/app.jar"]
```

### 동작 과정 시각화

```
┌──────────────────────────────────────┐
│ Stage 1 (builder)                    │
│   FROM maven:3.8.4-openjdk-11        │
│   - OS (100MB)                       │
│   - JDK 11 (200MB)                   │
│   - Maven (100MB)                    │
│   - 소스 복사 (50MB)                   │
│   - mvn package 실행                  │
│     → app.jar 생성 (10MB)             │
│ 총 크기: 460MB                         │
│ ⚠️  이 스테이지는 버려짐!                 │
└──────────────────────────────────────┘
                  │
                  │ COPY --from=builder
                  │ (app.jar만 복사)
                  ↓
┌──────────────────────────────────────┐
│ Stage 2 (final)                      │
│   FROM openjdk:17-jdk-slim           │
│   - OS (50MB)                        │
│   - JDK 17 (150MB)                   │
│   - app.jar (10MB) ← 복사됨            │
│ 총 크기: 210MB                         │
│ ✅ 이것만 최종 이미지로!                  │
└──────────────────────────────────────┘
```

### 핵심 명령어: `COPY --from`

```dockerfile
COPY --from=builder /app/target/app.jar /app.jar
     ─────────────  ────────────────────  ───────
          │                  │               │
          │                  │               └─ 최종 이미지 경로
          │                  └─ builder 스테이지의 파일 경로
          └─ 어느 스테이지에서 복사할지
```

Docker 처리 순서:

1. Stage 1 실행 → 빌드 → `app.jar` 생성
2. Stage 2 실행 → Stage 1에서 `app.jar`만 복사
3. Stage 1 전체는 **버림** (최종 이미지에 포함 안 됨)

---

## 7. 멀티스테이지 활용 패턴

### 패턴 1: 다른 이미지에서 특정 바이너리만 추출

JDK 버전이 다른 이미지에서 Maven만 가져오는 경우:

```dockerfile
# ===== Stage 1: Maven 소스 =====
FROM jdk11-maven:3.8.4 AS maven-source
# 이 이미지에 /usr/share/maven 있음

# ===== Stage 2: 최종 이미지 =====
FROM jdk17:latest
COPY --from=maven-source /usr/share/maven /usr/share/maven
ENV MAVEN_HOME=/usr/share/maven
ENV PATH=$MAVEN_HOME/bin:$PATH
```

동작 과정:

```
┌────────────────────────────────┐
│ Stage 1: maven-source          │
│   jdk11-maven:3.8.4            │
│   - JDK 11 (200MB)             │
│   - Maven 3.8.4 (10MB)         │
│   - 기타 (50MB)                 │
│ 총: 260MB  ⚠️ 이 전체는 버려짐!    │
└────────────────────────────────┘
         │
         │ /usr/share/maven만 복사 (10MB)
         ↓
┌────────────────────────────────┐
│ Stage 2: final                 │
│   jdk17:latest                 │
│   - JDK 17 (220MB)             │
│   - Maven 3.8.4 (10MB) ← 복사   │
│ 총: 230MB  ✅ JDK17 + Maven!    │
└────────────────────────────────┘
```

### 패턴 2: HyperCloud CI/CD 실무 예시

```dockerfile
# Tekton Pipeline에서 사용하는 Dockerfile
# ===== Stage 1: 빌드 =====
FROM hyperregistry.domain/maven:3.8.4-jdk11 AS builder
COPY . /workspace
WORKDIR /workspace
RUN mvn clean package -DskipTests

# ===== Stage 2: 실행 =====
FROM hyperregistry.domain/openjdk:17-jre-slim
COPY --from=builder /workspace/target/*.jar /app/app.jar

HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

HyperRegistry에 푸시:

```bash
docker build -t hyperregistry.domain/myapp:1.0 .
docker push hyperregistry.domain/myapp:1.0
```

결과:

```
빌드 이미지 (builder): 450MB → 버려짐
최종 이미지 (myapp):   180MB → HyperRegistry에 저장
```

---

## 8. 멀티스테이지 빌드 실제 빌드 확인

```bash
docker build -t test:latest .
```

출력 예시:

```
Step 1/5 : FROM jdk11-maven:3.8.4 AS maven-source
 ---> a1b2c3d4e5f6
Step 2/5 : FROM jdk17:latest
 ---> f6e5d4c3b2a1
Step 3/5 : COPY --from=maven-source /usr/share/maven /usr/share/maven
 ---> Running in 1234567890ab
Step 4/5 : ENV MAVEN_HOME=/usr/share/maven
Step 5/5 : ENV PATH=$MAVEN_HOME/bin:$PATH
Successfully built d4c3b2a1f6e5
```

최종 이미지 크기 확인:

```bash
docker images
```

```
REPOSITORY    TAG       SIZE
test          latest    230MB   # 최종 이미지 (JDK17 + Maven만)
jdk11-maven   3.8.4     260MB   # Stage 1 (최종 이미지에 미포함)
jdk17         latest    220MB   # Stage 2 베이스
```

---

## 9. 멀티스테이지 빌드의 장점

**이미지 크기 감소**:

```
단일 스테이지: 500MB (빌드 도구 + 결과물)
멀티 스테이지: 210MB (결과물만)
```

**보안 강화**: 빌드 도구와 소스코드가 최종 이미지에 없으므로 공격 표면 감소

**레이어 관리**: 필요한 것만 포함되어 컨테이너 시작 속도 향상

---

## 10. 핵심 정리

```
멀티스테이지 = 여러 단계로 나눠서 필요한 것만 가져오기

Stage 1 (builder)        Stage 2 (final)
┌───────────┐           ┌───────────┐
│ 빌드 환경   │──복사────→ │ 실행 환경    │
│ (전체)     │           │ (최소)     │
└───────────┘           └───────────┘
     ↓                        ↓
  버려짐                  최종 이미지
```

**`COPY --from=스테이지명`** 이 멀티스테이지 빌드의 핵심입니다.