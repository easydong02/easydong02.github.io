---
title: "[Docker] 가상머신에서 도커 설치와 실행"
date: 2024-08-16 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, vm, install, run, image]
render_with_liquid: false
---

## ✅도커 설치부터 실행까지, 막힘없이 따라 해보자!

지난 포스팅에서 컨테이너 기술의 놀라운 장점들을 살펴봤는데요, 이제 컨테이너 기술의 대표 주자인 **Docker**를 직접 설치하고 실행해보는 시간을 가져보겠습니다.

### ☑️도커 설치하기

**1. 시스템 요구 사항 확인**

- **리눅스 커널 버전:** 3.10 이상
- **아키텍처:** 64비트 (x86_64)

**2. 도커 설치 (Ubuntu)**

- **apt 업데이트:** `sudo apt update`
- **필요 패키지 설치:** `sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common`
- **Docker 공식 GPG 키 추가:** `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`
- **apt 저장소에 Docker 저장소 추가:** `echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
- **apt 업데이트:** `sudo apt update`
- **Docker CE 설치:** `sudo apt -y install docker-ce`
- **사용자 등록:** `sudo usermod -aG docker $USER`
- **Docker 서비스 활성화 및 재시작:** `sudo systemctl daemon-reload`, `sudo systemctl enable docker`, `sudo systemctl restart docker`

**3. 도커 설치 (CentOS)**

- **Docker 저장소 추가:** `yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
- **Docker CE 설치 및 업데이트:** `yum -y install docker-ce && yum -y update`
- **Docker 서비스 활성화 및 시작:** `systemctl daemon-reload`, `systemctl enable --now docker`, `systemctl start docker`

**4. 기타 설치 방법**

- **셸 스크립트를 이용한 자동 설치:** `curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh`
- **수동 설치:** Docker 공식 문서를 참고하여 필요한 패키지를 직접 다운로드하고 설치할 수 있습니다.

### ☑️도커 실행하고 이미지 가져오기

**1. 도커 정보 확인:**

- `docker info` 명령을 사용하여 설치된 Docker의 정보를 확인합니다.

**2. 도커 이미지 가져오기 (Pull)**

- `docker pull [이미지 이름]:[태그]` 명령을 사용하여 Docker Hub에서 원하는 이미지를 가져옵니다.
- 예시: `docker pull centos:7`, `docker pull ubuntu:16.04`

**3. 도커 컨테이너 실행 (Run)**

- `docker run [옵션] [이미지 이름]:[태그] [명령어]` 명령을 사용하여 컨테이너를 실행합니다.
- 자주 사용되는 옵션:
    - `it`: 컨테이너에 인터랙티브 터미널 연결
    - `-name`: 컨테이너 이름 지정
    - `d`: 백그라운드 실행
    - `p`: 호스트와 컨테이너의 포트 연결

**4. 도커 컨테이너 관리**

- `docker ps`: 실행 중인 컨테이너 목록 확인
- `docker ps -a`: 모든 컨테이너 목록 확인
- `docker start [컨테이너 이름 또는 ID]`: 컨테이너 시작
- `docker stop [컨테이너 이름 또는 ID]`: 컨테이너 중지
- `docker restart [컨테이너 이름 또는 ID]`: 컨테이너 재시작
- `docker rm [컨테이너 이름 또는 ID]`: 컨테이너 삭제
- `docker exec -it [컨테이너 이름 또는 ID] [명령어]`: 실행 중인 컨테이너에 명령어 실행

### ☑️도커 이미지 살펴보기

- `docker images`: 로컬에 저장된 이미지 목록 확인
- `docker image history [이미지 이름]:[태그]`: 이미지의 생성 히스토리 확인

### ☑️웹 서버 컨테이너 실행하고 접속하기

- `docker pull nginx:1.25.0-alpine`: Nginx 웹 서버 이미지 가져오기
- `docker run -d -p 8001:80 --name=my-webserver1 nginx:1.25.0-alpine`: Nginx 컨테이너 실행 (호스트의 8001 포트를 컨테이너의 80 포트에 연결)
- `curl localhost:8001`: 웹 브라우저 또는 `curl` 명령을 사용하여 웹 서버 접속

### ☑️컨테이너 내부 파일 변경 및 이미지 생성

1. **컨테이너 내부 파일 변경:**
- `docker cp [호스트 파일 경로] [컨테이너 이름]:[컨테이너 파일 경로]`: 호스트에서 컨테이너로 파일 복사
- 예시: `docker cp index.html my-webserver1:/usr/share/nginx/html/index.html`
1. **Dockerfile 작성 및 이미지 생성:**
- `Dockerfile` 작성: 원하는 이미지 설정 및 파일 복사 명령어 작성
- `docker build -t [이미지 이름]:[태그] .`: Dockerfile을 기반으로 이미지 생성
- `docker run -d -p 8002:80 --name=my-webserver2 myweb:v1.0`: 생성한 이미지를 사용하여 컨테이너 실행

### ☑️데이터베이스 컨테이너 실행

- `docker pull mysql:5.7-debian`: MySQL 데이터베이스 이미지 가져오기
- `docker run -it -e MYSQL_ROOT_PASSWORD=pass123# mysql:5.7-debian /bin/bash`: MySQL 컨테이너 실행 (루트 비밀번호 설정)
- `docker exec -it [컨테이너 이름 또는 ID] /bin/bash`: 실행 중인 컨테이너에 접속하여 데이터베이스 관리

### ☑️Portainer로 컨테이너 관리하기

- `docker pull portainer/portainer-ce`: Portainer 이미지 가져오기
- `docker volume create portainer_data`: Portainer 데이터 저장용 볼륨 생성
- `docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --restart=always portainer/portainer-ce`: Portainer 컨테이너 실행 (호스트의 9000 포트에 연결)
- 웹 브라우저에서 `서버IP:9000`으로 접속하여 Portainer 사용

📌**참고:** 실습 환경 및 Docker 버전에 따라 일부 명령어나 설정이 다를 수 있습니다. 자세한 내용은 Docker 공식 문서를 참고하시기 바랍니다.