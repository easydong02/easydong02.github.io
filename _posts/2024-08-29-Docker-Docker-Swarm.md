---
title: "[Docker] 도커스웜(Docker Swarm)"
date: 2024-08-29 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, dockerswarm]
render_with_liquid: false
---

## ✅ Docker Compose로 멀티 컨테이너 애플리케이션, 한방에 관리!

지난 포스팅에서 Dockerfile을 통해 나만의 이미지를 만드는 방법을 알아보았습니다. 하지만 실제 애플리케이션은 여러 개의 컨테이너가 서로 유기적으로 연결되어 동작하는 경우가 많습니다. 이런 복잡한 멀티 컨테이너 환경을 효율적으로 관리하기 위해 **Docker Compose**가 등장했습니다. 이번 포스팅에서는 Docker Compose의 핵심 기능과 활용 방법을 자세히 알아보고, 실제 애플리케이션 배포까지 함께 해보겠습니다.

### ☑️ 왜 Docker Compose를 사용해야 할까요?

- **간편한 멀티 컨테이너 관리:** 여러 컨테이너를 개별적으로 생성하고 관리하는 것은 번거롭고 오류 발생 가능성도 높습니다. Docker Compose는 YAML 파일 하나로 여러 컨테이너를 정의하고, 단일 명령어로 컨테이너들을 한 번에 생성, 실행, 중지, 삭제할 수 있습니다.
- **컨테이너 간 의존성 관리:** 컨테이너 간의 의존성을 명시하여 특정 컨테이너가 실행되기 전에 필요한 다른 컨테이너가 먼저 실행되도록 할 수 있습니다. 예를 들어, 웹 서버 컨테이너가 실행되기 전에 데이터베이스 컨테이너가 먼저 실행되어야 합니다.
- **네트워크 및 볼륨 관리:** Docker Compose는 컨테이너들이 사용할 네트워크와 볼륨을 정의하고 관리할 수 있습니다. 컨테이너 간 통신을 위한 네트워크를 생성하고, 데이터를 영구적으로 저장하기 위한 볼륨을 설정할 수 있습니다.
- **환경 변수 관리:** 컨테이너에 필요한 환경 변수를 YAML 파일에 정의하여 쉽게 관리하고 변경할 수 있습니다.
- **확장성:** `scale` 또는 `deploy.replicas` 옵션을 사용하여 특정 서비스의 컨테이너 개수를 조절하여 애플리케이션의 부하를 분산하고 확장성을 높일 수 있습니다.

### ☑️ Docker Compose YAML 파일 작성하기

Docker Compose는 `docker-compose.yml`이라는 YAML 파일을 사용하여 멀티 컨테이너 애플리케이션을 정의합니다. YAML 파일은 사람이 읽기 쉽고 간결한 문법을 제공하며, 들여쓰기를 사용하여 계층 구조를 표현합니다.

📌**기본 구조**

`version: '3.8' # Docker Compose 파일 버전
services: # 컨테이너 서비스 정의
  서비스명1: # 컨테이너 서비스 이름
    image: 이미지 이름 # 사용할 이미지
    # ... 기타 컨테이너 설정
  서비스명2:
    # ...
networks: # 네트워크 설정 (선택 사항)
volumes: # 볼륨 설정 (선택 사항)`

📌**자주 사용되는 설정 옵션**

- `build`: 이미지를 빌드하기 위한 설정 (Dockerfile 경로 등)
- `image`: 사용할 이미지 이름
- `container_name`: 컨테이너 이름 지정
- `command`: 컨테이너 실행 시 실행할 명령어
- `depends_on`: 다른 서비스에 대한 의존성 설정
- `environment`: 환경 변수 설정
- `ports`: 호스트와 컨테이너의 포트 매핑
- `volumes`: 볼륨 마운트 설정
- `networks`: 컨테이너가 연결될 네트워크 설정
- `restart`: 컨테이너 재시작 정책 설정
- `deploy`: 컨테이너 배포 관련 설정 (replicas, resources 등)
- `healthcheck`: 컨테이너 상태 확인 설정

### ☑️ Docker Compose CLI 명령어

- `docker-compose up`: 컨테이너 생성 및 실행
- `docker-compose down`: 컨테이너 중지 및 삭제
- `docker-compose ps`: 실행 중인 컨테이너 목록 확인
- `docker-compose logs`: 컨테이너 로그 확인
- `docker-compose start`, `stop`, `restart`: 컨테이너 시작, 중지, 재시작
- `docker-compose build`: 이미지 빌드
- `docker-compose config`: YAML 파일 내용 확인
- `docker-compose scale`: 특정 서비스의 컨테이너 개수 조절

### ☑️ 3-Tier 웹 애플리케이션 배포 예시

Docker Compose를 사용하여 웹 프론트엔드, 백엔드 서버, 데이터베이스로 구성된 3-Tier 웹 애플리케이션을 배포해보겠습니다.

**1. 프로젝트 구조**

`my-app/
  docker-compose.yml
  frontend/
    Dockerfile
    ...
  backend/
    Dockerfile
    ...`

**2. `docker-compose.yml` 파일**

```bash
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:3000"
    networks:
      - frontend
    depends_on:
      - backend

  backend:
    build: ./backend
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: your_password
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db_data:
```

**3. 실행**

Bash

`docker-compose up --build -d`