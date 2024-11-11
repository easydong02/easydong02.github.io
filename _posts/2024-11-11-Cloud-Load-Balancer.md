---
title: "[Cloud] 로드밸런서(Load Balancer)의 이해"
date: 2024-11-11 00:00:00 +0900
categories: [클라우드 인프라 구축 프로젝트]
tags: [cloud, lb, loadbalancer, alb, nlb]
render_with_liquid: false
---

# AWS **LoadBalaner의 이해**

---

> **학습 목표
- 가용영역과 고가용성 서비스에 대해 이해합니다.
- 로드 밸런서의 역할을 이해합니다.
- AWS가 지원하는 로드밸런서의 종류를 확인합니다.
- Amazon EC2에서 로드밸런서를 생성하는 순서를 확인합니다.**
> 

---

### 가용영역과 고가용성 서비스

- 가용 영역(**Availability Zones)**
    - 가용 영역은 **물리적으로 격리된 데이터 센터 그룹**으로,
    - 각각이 독립된 전력, 냉각, 네트워크를 가지고 있는데,
    - AWS는 가용영역을 통해 **고가용성과 내결함성**을 제공
    - 리전마다 여러 개의 가용 영역을 가짐. 
    AWS 리전은 지리적 영역 내에서 **격리되고 물리적으로 분리된 *최소 2*개의 AZ로 구성**되어 있음.
    서울 리전은 4개의 가용영역, 미국동부 버지니아 북부는 6개, 오사카는 3개의 가용 영역을 포함.
        
        ![Untitled](/assets/img/Cloud/lb/Untitled.png)
        

- **고가용성(High Availability)**
    - 동일 목적의 서버를 2개 이상 운영하면서 각각 서로 다른 AZ에서 실행
    - 어느 한 AZ의 문제(정전 등)로 인해 서버가 다운되더라도 다른 AZ에서 동작중인 서버는 계속 작동하여 애플리케이션이 서비스 연속성을 보장함.
        
        ![Untitled](/assets/img/Cloud/lb/Untitled%201.png)
        

### AWS **로드 밸런싱(Load Balancing)**

- 여러 AZ에 서버가 있으면 어느 **한 서버에 트래픽이 집중되지 않도록 서버 간의 트래픽 부하 분산**해야 함.
- **로드 밸런서(Load Balancer)는 서버에 네트워크 트래픽을 분산시키는 장비**
- AWS에서는 2개 이상의 가용 영역에 동일한 서버를 배치하고, 로드밸런서를 통해 부하 분산
    
    ![Untitled](/assets/img/Cloud/lb/Untitled%202.png)
    

### AWS 로드밸런서 종류

- **애플리케이션 로드 밸런서(ALB):**
    - OSI 모델의 애플리케이션 계층(**7계층**)에서 작동하며**, HTTP 및 HTTPS 트래픽 분산에 적합**
    - 콘텐츠 기반 라우팅, 호스트 기반 라우팅, 경로 기반 라우팅과 같은 고급 기능을 제공
    - 복잡한 규칙 기반 라우팅이 필요한 애플리케이션과 마이크로서비스 아키텍처에 이상적
- **네트워크 로드 밸런서(NLB):**
    - OSI 모델의 전송 계층(**4계층**)에서 작동하며, TCP, UDP 및 TLS 트래픽에 가장 적합
    - **높은 성능과 낮은 대기 시간을 제공해서 일관된 성능과 네트워크 수준의 로드 밸런싱이 필요한 상황에 적합**
    
    ![Untitled](/assets/img/Cloud/lb/Untitled%203.png)
    

- **게이트웨이 로드 밸런서(GWLB):**
    - OSI 모델의 네트워크 계층(3계층)에서 작동하며, IP 패킷을 수신하고 수신자 규칙에 지정된 대상 그룹으로 트래픽을 전달
    - 다른 네트워크(VPC)의 **게이트웨이**역할로, 가상 어플라이언스로 라우팅할 수 있도록 해주는 서비스
    - 방화벽, 침입 탐지 및 방지 시스템, 심층 패킷 검사 시스템과 같은 가상 어플라이언스를 배포, 확장 및 관리에 적합함.
    - 게이트웨이 로드 밸런서는 게이트웨이 로드 밸런서 엔드포인트를 사용하여 VPC 경계 간에 트래픽을 안전하게 교환합니다.
    
    ![[https://support.bespinglobal.com/ko/support/solutions/articles/73000544791--aws-aws-gateway-load-balancer-gwlb-의-작동-방식-및-구성-방법](https://support.bespinglobal.com/ko/support/solutions/articles/73000544791--aws-aws-gateway-load-balancer-gwlb-%EC%9D%98-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9D-%EB%B0%8F-%EA%B5%AC%EC%84%B1-%EB%B0%A9%EB%B2%95)](/assets/img/Cloud/lb/Untitled%204.png)
    
    [https://support.bespinglobal.com/ko/support/solutions/articles/73000544791--aws-aws-gateway-load-balancer-gwlb-의-작동-방식-및-구성-방법](https://support.bespinglobal.com/ko/support/solutions/articles/73000544791--aws-aws-gateway-load-balancer-gwlb-%EC%9D%98-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9D-%EB%B0%8F-%EA%B5%AC%EC%84%B1-%EB%B0%A9%EB%B2%95)
    

### AWS 로드밸런서 생성 단계

- **대상 그룹(Target Group) 생성**
    - 트래픽을 어디로 보낼것인지 결정
    - 지정한 프로토콜과 포트 번호를 사용하여 **EC2 인스턴스 같은 하나 이상의 등록된 대상으로 요청을 라우팅합니다.**
- **로드밸런서 생성**
    - **리스너(Listener)** : 사용자가 구성한 포트 및 프로토콜을 사용하여 연결 요청을 검사. 
    리스너에 정의한 규칙에 따라 로드 밸런서가 등록된 대상으로 요청을 라우팅 합니다.
    
    ![Untitled](/assets/img/Cloud/lb/Untitled%205.png)
    

### LAB : ALB 구축하기

- 인프라 아키텍처
    
    ![image.png](/assets/img/Cloud/lb/image.png)
    

### 1. VPC 만들기

- 이름:  **lab-vpc**(10.0.0.0/16)
    
    ![image.png](/assets/img/Cloud/lb/image%201.png)
    
- Subnet
    - public 2 : 10.0.0.0/24, 10.0.1.0/24
    - private 2 : 10.0.2.0/24, 10.0.3.0/24

### 2. 보안 그룹 만들기

- ALB-SG
    - VPC : lab-vpc
    - 인바운드 규칙 : 80 ← anywhere
- WEB-SG
    - VPC : lab-vpc
    - 인바운드 규칙: 80 ← ALB-SG
- 결과
    
    ![image.png](/assets/img/Cloud/lb/image%202.png)
    

### 3. 웹 서버 만들기

- **web1**
    - AMI : Amazon Linux 2023 AMI
    - 인스턴스 유형: t2.micro
    - 키페어 : lab-key
    - 네트워크: Lab VPC
        - 서브넷:  lab-subnet-**private1-**ap-northeast-2a
        - 퍼블릭  IP:  비활성화
        - 보안그룹 : WEB-SG
    - 스토리지: 8G
    - 사용자 데이터
        
        ```bash
        #!/bin/bash 
        yum install httpd -y
        systemctl start httpd
        systemctl enable httpd
        echo "<h1>WEB1</h1>" > /var/www/html/index.html
        ```
        
- web2
    - AMI : Amazon Linux 2023 AMI
    - 인스턴스 유형: t2.micro
    - 키페어 : lab-key
    - 네트워크: Lab VPC
        - 서브넷:  lab-subnet-private2-ap-northeast-2c
        - 퍼블릭  IP:  비활성화
        - 보안그룹 : WEB-SG
    - 스토리지: 8G
    - 사용자 데이터
        
        ```bash
        #!/bin/bash 
        yum install httpd -y
        systemctl start httpd
        systemctl enable httpd
        echo "<h1>WEB2</h1>" > /var/www/html/index.html
        ```
        

### **4. Application Load Balancer 생성**

- 동작 아키텍처
    
    ![image.png](/assets/img/Cloud/lb/image%203.png)
    

- **대상 그룹 만들기**
    - 1단계 : 그룹 세부 정보 지정
        - 유형 : 인스턴스
        - 대상 그룹 이름 : **web-alb-tg**
        - 프로토콜/포트 : **HTTP/80**
        - VPC : **lab-vpc**
        - 프로토콜 버전 : **HTTP1**
        - 상태 검사
            - 프로토콜 : **HTTP**
            - 상태 검사 경로 : **/**
        - [다음]
            
            ![image.png](/assets/img/Cloud/lb/image%204.png)
            
        
    - 2단계 : 대상 등록
        - 사용 가능한 인스턴스
            - **web1**
            - **web2**
        - 선택한 인스턴스를 위한 포트 : 80
        - [아래에 보류 중인 것으로 포함 선택 후 대상 그룹 생성] 클릭
            
            ![image.png](/assets/img/Cloud/lb/image%205.png)
            
        - [대상그룹 생성]
            
            ![image.png](/assets/img/Cloud/lb/image%206.png)
            
        
- **로드 밸런서 만들기**
    - EC2 > 로드 밸런서 > [로드 밸런서 생성]
    - 기본 구성
        - 이름 : **web-alb**
        - 체계 : 인터넷 경계
        - IP 주소 유형 : IPv4
    - 네트워크 매핑
        - VPC : **lab-vpc**
        - 매핑 가용 영역/서브넷(※ **ALB가 있을 위치 지정**-퍼블릭 서브넷에 있어야 외부 연결 가능)
            - **ap-northeast-2a (apne2-az1), lab-subnet-public1-ap-northeast-2a**
            - **ap-northeast-2c (apne2-az3), lab-subnet-public2-ap-northeast-2a**
    - 보안 그룹 : **ALB-SG**
    - 리스너 및 라우팅
        - 프로토콜 : HTTP
        - 포트 : 80
        - 대상 그룹 : **web-alb-tg**
            
            ![image.png](/assets/img/Cloud/lb/image%207.png)
            
        
    - [로드 밸런서 생성]
        
        ![image.png](/assets/img/Cloud/lb/image%208.png)
        

### 5. 동작 확인

- ALB의 리소스 맵 확인
    
    ![image.png](/assets/img/Cloud/lb/image%209.png)
    

- ALB의 DNS 이름으로 접속 요청
    - ALB의 DNS 이름 확인

### 6. 리소스 삭제

- ALB
- 대상그룹
- Web1, Web2
- NAT - vpc
- 보안그룹
    - WEB-SG
    - ALB-SG