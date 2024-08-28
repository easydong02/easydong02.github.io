---
title: "[Docker] 도커파일(Dockerfile)"
date: 2024-08-28 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, dockerfile, image]
render_with_liquid: false
---

## ✅ Dockerfile: 나만의 맞춤형 이미지를 만들어보자!

지난 포스팅에서는 도커 이미지를 효율적으로 관리하는 다양한 방법을 살펴보았습니다. 이제는 도커 이미지를 직접 만들어보는 시간입니다! 도커 이미지는 `Dockerfile`이라는 파일을 통해 정의되고 생성됩니다. 이번 포스팅에서는 `Dockerfile`의 기본 개념부터 다양한 명령어, 그리고 효율적인 이미지 생성을 위한 최적화 기법까지 자세히 알아보겠습니다.

### ☑️ Dockerfile: 이미지 생성의 설계도

`Dockerfile`은 도커 이미지를 생성하기 위한 일련의 명령어들을 담고 있는 텍스트 파일입니다. 마치 건축 설계도처럼, `Dockerfile`은 이미지의 기반이 되는 베이스 이미지, 필요한 패키지 설치, 파일 복사, 환경 변수 설정, 실행 명령 등 이미지 생성에 필요한 모든 정보를 포함합니다.

`Dockerfile`을 작성하고 `docker build` 명령어를 실행하면 도커는 `Dockerfile`의 명령어들을 순차적으로 실행하여 이미지를 생성합니다. 이렇게 생성된 이미지는 Docker Hub와 같은 저장소에 업로드하여 다른 사람들과 공유하거나, 로컬 환경에서 컨테이너를 실행하는 데 사용할 수 있습니다.

### ☑️ Dockerfile 명령어: 이미지 생성의 핵심 도구

`Dockerfile`에는 다양한 명령어를 사용하여 이미지 생성 과정을 정의할 수 있습니다. 몇 가지 자주 사용되는 명령어들을 살펴보겠습니다.

- `FROM`: 베이스 이미지를 지정합니다. 모든 Dockerfile은 반드시 `FROM` 명령어로 시작해야 합니다.
- `RUN`: 이미지 내부에서 명령어를 실행합니다. 주로 패키지 설치, 파일 다운로드, 환경 설정 등에 사용됩니다.
- `COPY`: 호스트 시스템의 파일 또는 디렉토리를 이미지 내부로 복사합니다.
- `ADD`: `COPY`와 유사하지만, 추가적으로 압축 파일 자동 해제, URL 다운로드 등의 기능을 지원합니다.
- `CMD`: 컨테이너 실행 시 기본적으로 실행될 명령어를 지정합니다.
- `ENTRYPOINT`: 컨테이너 실행 시 항상 실행될 명령어를 지정합니다.
- `ENV`: 환경 변수를 설정합니다.
- `EXPOSE`: 컨테이너가 사용할 포트를 지정합니다.
- `VOLUME`: 컨테이너 데이터를 저장할 볼륨을 지정합니다.
- `USER`: 컨테이너 내부에서 명령을 실행할 사용자를 지정합니다.
- `WORKDIR`: 작업 디렉토리를 설정합니다.

### ☑️ 멀티 스테이지 빌드: 이미지 크기 줄이기

멀티 스테이지 빌드는 여러 개의 `FROM` 명령어를 사용하여 여러 단계로 이미지를 생성하는 기능입니다. 이를 통해 불필요한 파일이나 의존성을 제거하여 이미지 크기를 줄이고, 보안성을 향상시킬 수 있습니다.

예를 들어, 애플리케이션 빌드 단계와 실행 단계를 분리하여 빌드 도구나 중간 생성 파일들을 최종 이미지에 포함하지 않도록 할 수 있습니다.

**멀티 스테이지 빌드 예시 (Go 언어)**

`FROM golang:1.15-alpine3.12 AS gobuilder-stage
WORKDIR /app/
COPY gostart.go /app/
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /app/gostart

FROM scratch
COPY --from=gobuilder-stage /app/gostart /app/gostart
CMD ["/app/gostart"]`

### ☑️ 다양한 이미지 빌드 실습

`Dockerfile`을 사용하여 Python, Node.js, Java, Django 등 다양한 환경의 이미지를 직접 빌드해보고, 이미지 크기를 줄이기 위한 다양한 최적화 기법들을 실습해 볼 수 있습니다. 또한, Nexus를 활용하여 프라이빗 레지스트리를 구축하고 이미지를 안전하게 관리하는 방법도 배울 수 있습니다.

📌**간단한 웹 서버 이미지 예시 (Nginx)**

`FROM ubuntu:20.04
RUN apt -y update && apt install -y -q nginx
COPY index.html /var/www/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]`

📌**Flask 웹 애플리케이션 이미지 예시**

`FROM python:3.9-slim
WORKDIR /app
COPY requirement.txt .
COPY app/static static
COPY app/template template
COPY app.py .
RUN pip install --no-cache-dir -r requirement.txt
CMD ["python", "app.py"]`