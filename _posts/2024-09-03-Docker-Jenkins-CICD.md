---
title: "[Docker] 컨테이너 배포 자동화(feat. Jenkins)"
date: 2024-09-03 00:00:00 +0900
categories: [Infra, Docker]
tags: [docker, cicd, jenkins, container]
render_with_liquid: false
---

## ✅컨테이너 배포 자동화: AWS 서비스 기반 3-Tier CI/CD MSA 구축

이전 포스팅에서는 VM 기반 환경에서 3-Tier 컨테이너 애플리케이션의 CI/CD를 구성해보았습니다. 이번에는 AWS 서비스를 활용하여 좀 더 확장성 있고 안정적인 CI/CD 환경을 구축하는 방법을 알아보겠습니다. AWS CodeCommit, SNS, SQS, ECR 등 다양한 서비스를 활용하여 3-Tier 컨테이너 애플리케이션의 배포를 자동화하는 방법을 단계별로 살펴보겠습니다.

### ☑️1단계: AWS Cloud9 환경에 Jenkins 설치하기

AWS Cloud9은 클라우드 기반 통합 개발 환경(IDE)으로, 코드 작성, 실행, 디버깅 등 개발에 필요한 모든 기능을 제공합니다. 먼저 Cloud9 환경에 Jenkins를 설치하여 CI/CD 파이프라인을 구축할 준비를 해 보겠습니다.

- **Dockerfile 및 docker-compose.yaml 파일 생성**

Dockerfile

`FROM jenkins/jenkins:lts
USER root
RUN apt-get update && \
apt-get -y install apt-transport-https \
ca-certificates \
curl \
gnupg2 \
zip \
unzip \
software-properties-common && \
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
apt-get update && \
apt-get -y install docker-ce`

YAML

`version: '3.3'
services:
  jenkins:
    build:
      context: .
    container_name: jenkins
    user: root
    restart: always
    ports:
      - 18080:8080
      - 50000:50000
    volumes:
      - /home/ec2-user/jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock`

- **Jenkins 컨테이너 실행**

Bash

`docker compose up -d`

- **Jenkins 접속**

Cloud9 EC2 인스턴스의 퍼블릭 IP와 Jenkins 포트(18080)를 사용하여 웹 브라우저에서 Jenkins에 접속합니다. (예: http://15.164.237.24:18080)

- **Jenkins 초기 설정 및 플러그인 설치**

Jenkins 초기 설정 과정을 진행하고, 필요한 플러그인(Git, Pipeline, AWS 등)을 설치합니다. 자세한 설정 방법은 이전 포스팅(VM 기반 컨테이너 배포 자동화)을 참고해 주세요.

### ☑️2단계: 3-Tier 애플리케이션 배포 및 테스트

이전 실습에서 사용했던 3-Tier 애플리케이션(my-diary-3)을 가져와서 Jenkins를 통해 배포하고 테스트해 보겠습니다.

- **애플리케이션 소스 코드 가져오기 및 컨테이너 실행**

Bash

`cp -r /home/kevin/fastcampus/ch10/my-diary-3 .
cd my-diary-3
docker compose up -d`

코드를 사용할 때는 [주의](https://www.notion.so/faq#coding)가 필요합니다.

- **애플리케이션 접속 테스트**

웹 브라우저에서 `http://퍼블릭IP:3000`에 접속하여 애플리케이션이 정상적으로 실행되는지 확인합니다.

### ☑️3단계: CodeCommit 저장소 생성 및 연동

AWS CodeCommit은 안전하고 확장 가능한 프라이빗 Git 저장소를 제공하는 서비스입니다. 이제 CodeCommit 저장소를 생성하고, 애플리케이션 소스 코드를 저장소에 업로드해 보겠습니다.

- **CodeCommit 저장소 생성:** AWS 관리 콘솔에서 CodeCommit 서비스로 이동하여 `mydiary-repo`라는 이름의 저장소를 생성합니다.
- **IAM 사용자 생성 및 권한 부여:** IAM(Identity and Access Management) 서비스에서 Git 사용자를 생성하고, CodeCommit에 대한 권한을 부여합니다.
- **로컬 저장소와 CodeCommit 연결:**

Bash

`git clone https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/mydiary-repo`

코드를 사용할 때는 [주의](https://www.notion.so/faq#coding)가 필요합니다.

- **소스 코드 업로드:** 애플리케이션 소스 코드를 복사하여 CodeCommit 저장소에 커밋하고 푸시합니다.

### ☑️4단계: SNS, SQS, CodeCommit Trigger 설정

- **SNS 토픽 생성:** Amazon SNS(Simple Notification Service) 콘솔에서 `mydiary-notification`이라는 이름의 토픽을 생성합니다.
- **SQS 대기열 생성:** Amazon SQS(Simple Queue Service) 콘솔에서 `mydiary-event-queue`라는 이름의 대기열을 생성합니다.
- **CodeCommit 트리거 생성:** CodeCommit 저장소 설정에서 트리거를 생성하고, 이벤트 유형을 `푸시`로, SNS 토픽을 `mydiary-notification`으로 설정합니다.
- **SNS 구독 생성:** `mydiary-notification` 토픽에 대한 구독을 생성하고, 프로토콜을 `SQS`, 엔드포인트를 `mydiary-event-queue`로 설정합니다.

### ☑️5단계: Jenkins 설정 및 파이프라인 생성

- **Jenkins Credentials 설정:**
    - AWS 자격 증명: AWS 액세스 키 ID 및 비밀 액세스 키를 Jenkins Credentials에 추가합니다.
    - Docker Hub 자격 증명: Docker Hub 사용자 이름과 비밀번호 또는 토큰을 Jenkins Credentials에 추가합니다.
- **Jenkins Pipeline 생성:**
    - 새로운 아이템 생성 후, "Pipeline" 유형을 선택합니다.
    - Pipeline 이름을 지정하고, "Pipeline script from SCM" 옵션을 선택합니다.
    - SCM: Git
    - Repository URL: CodeCommit 저장소 URL
    - Credentials: CodeCommit 연결 시 생성한 자격 증명
    - Branch Specifier: `/master`
    - Script Path: `Jenkinsfile`
- **Jenkinsfile 작성**

`pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t mydiary-repo .'
            }
        }
        stage('Tag') {
            steps {
                sh 'docker tag mydiary-repo:latest 594682333406.dkr.ecr.ap-northeast-2.amazonaws.com/mydiary-repo:1.0'
            }
        }
        stage('Push') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('awsaccess')
                AWS_SECRET_ACCESS_KEY = credentials('awssecret')
            }
            steps {
                sh 'aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 594682333406.dkr.ecr.ap-northeast-2.amazonaws.com'
                sh 'docker push 594682333406.dkr.ecr.ap-northeast-2.amazonaws.com/mydiary-repo:1.0'
            }
        }
    }
}`

### ☑️6단계: 자동 배포 테스트

- **소스 코드 변경 및 푸시:** `public/index.ejs` 파일을 수정하고, CodeCommit 저장소에 커밋하고 푸시합니다.
- **자동 배포 확인:** Jenkins에서 빌드가 실행되고, ECR에 이미지가 푸시되는지 확인합니다.
- **애플리케이션 접속:** 웹 브라우저에서 애플리케이션에 접속하여 변경 사항이 반영되었는지 확인합니다.