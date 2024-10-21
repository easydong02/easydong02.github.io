---
title: "[Cloud] 클라우드 컴퓨팅의 이해"
date: 2024-10-18 00:00:00 +0900
categories: [클라우드 인프라 구축 프로젝트]
tags: [kubernetes, webapp]
render_with_liquid: false
---

## 클라우드 컴퓨팅의 이해

---

- **학습 목표**
    - 클라우드 컴퓨팅 소개
    - 클라우드 컴퓨팅의 이점
    - AWS 글로벌 인프라

---

### 1. 클라우드 컴퓨팅의 정의

- **클라우드** 은 컴퓨팅 파워, 데이터베이스, 스토리지, 애플리케이션 및 기타 IT 리소스를 **온디맨드**로 **인터넷을** **통해** 제공하고 **사용한** **만큼만** **비용을** **지불**하는 것을 말합니다.
    
    ![image.png](/assets/img/Cloud/cloud-computing/image.png)
    

- 물리적 데이터 센터와 서버를 구입, 소유 및 유지 관리하는 대신, Amazon Web Services(AWS)와 같은 **클라우드** **공급자(CSP)** 로부터 필요에 따라 컴퓨팅 파워, 스토리지, 데이터베이스와 같은 기술 서비스에 액세스할 수 있습니다

### **2. IT 서비스** **모델**

- 서비스 모델
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%201.png)
    

### **3. 클라우드** **컴퓨팅** **배포** **모델**

- 컴퓨팅 모델
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%202.png)
    

### 4. 클라우드 컴퓨팅의 이점

- **자본** **비용(CapEx)을 가변** **비용으로** **대체**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%203.png)
    
- **거대한** **규모의** **경제 (Massive economies of scale)**
    - 모든 고객으로부터 모은 사용량 덕분에 AWS는 고객을 대상으로 더 높은 수준의 **규모의** **경제**를 실현하고 **비용** **절감의** **혜택**을 고객들에게 돌려줍니다.
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%204.png)
    

- **용량** **추정** **불필요 (Stop guessing capacity)**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%205.png)
    

- **속도 및 민첩성** **향상(Increase speed and agility)**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%206.png)
    

- **데이터** **센터** **운영 및 유지** **관리에** **비용** **투자** **불필요**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%207.png)
    
- **몇 분 만에 전 세계에** **배포 (Go global in minutes)**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%208.png)
    

### 5. AWS 글로벌 인프라

- **AWS 글로벌** **인프라 (AWS Global Infrastructure)**
    - **AWS 글로벌** **인프라**는 우수한 품질의 **글로벌** **네트워크** **성능**으로 **유연하고**, **안정적이며**, **확장** **가능하고**, **안전한** 클라우드 컴퓨팅 환경을 제공하도록 설계되고 구축되었습니다.
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%209.png)
    

- **AWS 리전** **(Region)**
    - Amazon 클라우드 컴퓨팅 리소스는 전 세계 여러 위치에서 호스팅. 이러한 위치가 AWS 리전입니다.
        - **AWS 리전**은 지리적 영역으로 전 세계에 총 [34개의 리전](https://aws.amazon.com/ko/about-aws/global-infrastructure/)이 있습니다.
    - 내결함성 및 안정성을 최대한 높이기 위해 각 리전은 서로 완전히 분리되고, 이중화 되도록 설계
        - 리전 간의 데이터 복제는 고객이 제어합니다.
        - 리전 간 통신에는 AWS 백본 네트워크 인프라가 사용됩니다.
    - 리전은 일반적으로 2개 이상의 **가용** **영역**으로 구성됩니다.
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%2010.png)
    

- **[참고] 리전** **선택시** **고려** **사항**
    
    ![image.png](/assets/img/Cloud/cloud-computing/image%2011.png)
    

- **가용** **영역 (Availability Zones)**
    - 각 **AWS 리전**에는 **여러** **개의** **가용** **영역**이 있습니다.
    - 각 **가용** **영역**은 **물리적으로** **분리된** 별개의 **여러** **데이터** **센터**로 구성됩니다.
        - 현재 전 세계에는 [108개의 가용](https://aws.amazon.com/ko/about-aws/global-infrastructure/) [영역](https://aws.amazon.com/ko/about-aws/global-infrastructure/)이 있습니다.
        - 물리적으로 분리되어, **내결함성을** **제공**. 일본과 같은 지진 발생 지역에서는 각 가용 영역이 서로 다른 전력망, 범람원, 단층선에 배치되어 있습니다.
        - **고속** **프라이빗** **네트워크**를 통해 다른 가용 영역과 상호 연결되므로 동기 복제가 가능
        - 사용할 가용 영역을 **고객이** **선택**합니다.
    - 복원력을 위해 **여러** **가용** **영역에** **걸쳐** **데이터와** **리소스를** **복제하는** **것이** **좋습니다**.
        
        ![image.png](/assets/img/Cloud/cloud-computing/image%2012.png)