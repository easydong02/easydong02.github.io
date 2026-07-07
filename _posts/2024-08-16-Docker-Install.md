---
title: "[Docker] 가상머신에서 도커 설치와 실행"
date: 2024-08-16 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, container, vm, install, run, image]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **도커를 직접 설치하고 실행**하는 전 과정을 정리한다. Ubuntu·CentOS 설치부터 컨테이너 실행·관리 명령어, 웹 서버·DB 컨테이너 띄우기, 그리고 GUI 관리 도구 **Portainer**까지 실습한다.

> **시작 전 요구 사항** — 리눅스 **커널 3.10 이상**, **64비트(x86_64)** 아키텍처가 필요하다.

---

## 1. 도커 설치

### Ubuntu

```bash
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common

# Docker 공식 GPG 키 + 저장소 추가
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt -y install docker-ce
sudo usermod -aG docker $USER        # 사용자를 docker 그룹에 등록
sudo systemctl enable --now docker
```

### CentOS

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum -y install docker-ce && yum -y update
systemctl enable --now docker
```

### 간편 설치 (스크립트)

```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
```

> 💡 `sudo usermod -aG docker $USER`로 사용자를 **docker 그룹에 등록**하면, 매번 `sudo` 없이 도커 명령을 쓸 수 있다(적용하려면 재로그인 필요).

---

## 2. 컨테이너 실행 & 관리 명령어

이미지를 가져와(`pull`) 컨테이너로 실행(`run`)한다.

```bash
docker info                                # 설치 정보 확인
docker pull ubuntu:16.04                    # 이미지 가져오기
docker run [옵션] [이미지]:[태그] [명령어]     # 컨테이너 실행
```

**`run` 주요 옵션:**

| 옵션 | 의미 |
|:---:|------|
| `-it` | 인터랙티브 터미널 연결 |
| `--name` | 컨테이너 이름 지정 |
| `-d` | 백그라운드(detached) 실행 |
| `-p` | 호스트:컨테이너 포트 연결 |

**컨테이너 관리:**

| 명령 | 역할 |
|------|------|
| `docker ps` / `ps -a` | 실행 중 / 전체 컨테이너 목록 |
| `docker start/stop/restart` | 시작 / 중지 / 재시작 |
| `docker rm` | 컨테이너 삭제 |
| `docker exec -it <컨테이너> <명령>` | 실행 중 컨테이너에 명령 실행 |
| `docker images` | 로컬 이미지 목록 |

---

## 3. 웹 서버 컨테이너 실행

Nginx 이미지를 받아 **호스트 8001 → 컨테이너 80**으로 연결해 실행한다.

```bash
docker pull nginx:1.25.0-alpine
docker run -d -p 8001:80 --name=my-webserver1 nginx:1.25.0-alpine
curl localhost:8001                          # 접속 확인
```

> 💡 `-p 8001:80`은 **호스트의 8001 포트로 온 요청을 컨테이너의 80 포트로 전달**한다는 뜻이다. 이 포트 매핑이 있어야 외부에서 컨테이너 안의 서비스에 접근할 수 있다.

---

## 4. 파일 변경 & 커스텀 이미지 생성

호스트 파일을 컨테이너로 복사하거나, **Dockerfile로 이미지를 빌드**한다.

```bash
# 호스트 → 컨테이너 파일 복사
docker cp index.html my-webserver1:/usr/share/nginx/html/index.html

# Dockerfile 기반 이미지 빌드 & 실행
docker build -t myweb:v1.0 .
docker run -d -p 8002:80 --name=my-webserver2 myweb:v1.0
```

---

## 5. DB 컨테이너 & Portainer

**MySQL 컨테이너** — 환경 변수(`-e`)로 루트 비밀번호를 설정한다.

```bash
docker pull mysql:5.7-debian
docker run -it -e MYSQL_ROOT_PASSWORD=pass123# mysql:5.7-debian /bin/bash
docker exec -it <컨테이너> /bin/bash          # 접속해 DB 관리
```

**Portainer** — 컨테이너를 웹 GUI로 관리하는 도구.

```bash
docker pull portainer/portainer-ce
docker volume create portainer_data
docker run -d -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data --restart=always portainer/portainer-ce
# 브라우저에서 서버IP:9000 접속
```

> 💡 Portainer 실행 시 **`/var/run/docker.sock`을 마운트**하는 이유는, 컨테이너 안의 Portainer가 **호스트의 도커 엔진을 제어**하기 위해서다. 이 소켓이 도커 데몬과 통신하는 창구다.

---

## 📝 정리

```
도커 설치 & 실행
├─ 설치   apt/yum 저장소 또는 get.docker.com 스크립트
├─ 실행   pull → run (-it/-d/-p/--name)
├─ 관리   ps / start·stop / exec / rm
├─ 서비스 nginx(-p 8001:80), mysql(-e 비밀번호)
└─ 관리툴 Portainer(docker.sock 마운트)
```

| 명령 | 역할 |
|------|------|
| `run -d -p` | 백그라운드 + 포트 매핑 실행 |
| `exec -it` | 실행 중 컨테이너 접속 |
| `docker cp` | 호스트 ↔ 컨테이너 파일 복사 |

도커 실행의 핵심은 **`pull → run`**과 **포트 매핑(`-p`)**이다. 이 기본기를 잡으면 웹 서버·DB 등 어떤 서비스든 컨테이너로 띄울 수 있고, Portainer로 이들을 시각적으로 관리할 수 있다.
