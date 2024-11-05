---
title: "[Cloud] AWS 관리서버(Bastion)의 이해"
date: 2024-11-05 00:00:00 +0900
categories: [클라우드 인프라 구축 프로젝트]
tags: [cloud, bastion, server, ec2]
render_with_liquid: false
---

# AWS 관리 서버(Bastion) 이해

---

> **학습 목표**
- Bastion 서버의 역할을 이해합니다.
- Bastion 동작 방식을 확인합니다.
> 

---

## Bastion Server의 역할

- **점프 서버(Jump Server)**라고도 알려진 Bastion Server는 **공격을 견딜 수 있도록 특별히 설계 및 구성된 네트워크의 특수 목적 컴퓨터(인스턴스)**

![[[원문](https://jongroinf.com/news_Cloud_Docker/30593)]](/assets/img/Cloud/bastion/image.png)

[[원문](https://jongroinf.com/news_Cloud_Docker/30593)]

- 퍼블릭 서브넷에  배치해서, 서버 관리자가 프라이빗 서브넷에 존재하는 WEB이나 WES 서버에 액세스하기 위한 게이트웨이 역할을 수

![Untitled](/assets/img/Cloud/bastion/Untitled.png)

![Untitled](/assets/img/Cloud/bastion/Untitled%201.png)

- 보안그룹
    - Bastion-SG
        - 22 ← 내IP
    - application-SG
        - 22 ←Bastion-SG
- Bastion-server
    - 퍼블릭IP
    - public-subnet
    - Bastion-SG
        - PC(private-key.pem) —원격 로그인 → Bation-Server
- application-server
    - 퍼블릭IP X
    - private-subnet
    - application-SG
        - Bastion login → ~/.ssh/private-key.pem 파일 복사. 400퍼미션 조절 →원격로그인 → application-server