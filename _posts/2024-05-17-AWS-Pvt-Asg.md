---
title: "[AWS] Private Architecture(프라이빗 서브넷으로 오토 스케일링 + 동적 조정 정책)"
date: 2024-05-17 00:00:00 +0900
categories: [Infra, AWS]
tags: [aws, autoscaling, autopolicy, privatesubnet]
render_with_liquid: false
---

## ✅Private Subnet?

---

![Untitled](/assets/img/Infra/AWS/private/Untitled.png)

프라이빗 서브넷을 만들고 거기에 있는 인스턴스를 ALB를 통해서 접근하는 아키텍처를 구성하겠습니다.

## ✅Private Network 아키텍처 구성

---

### ☑️Private Subnet 생성

![Untitled](/assets/img/Infra/AWS/private/Untitled%201.png)

VPC - 서브넷 탭에서 프라이빗 서브넷을 만들겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%202.png)

4개의 가용 영역에서 각각의 프라이빗 서브넷을 만듭니다. CIDR 블록은 이미 만들어진 퍼블릿 서브넷 이후에 16비트를 더한 수부터 시작해서 만들었습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%203.png)

생성을하면 이런식으로 서브넷 리스트에서 볼 수 있습니다.

### ☑️Routing Table 생성

![Untitled](/assets/img/Infra/AWS/private/Untitled%204.png)

VPC - 라우팅 테이블 탭에서 프라이빗 서브넷 내에서 라우팅 로직을 참고하는 테이블을 만들겠습니다. 이게 있어야 프라이빗 서브넷에 있는 인스턴스에 접근이 가능합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%205.png)

생성된 라우팅 테이블을 클릭하여 들어가면 내부 통신을 위한 기본 로컬 라우팅이 존재합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%206.png)

여기에 연결 없는 서브넷 리스트 탭에서 아까만든 서브넷들이 보입니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%207.png)

검색 후 연결을 합니다.

### ☑️시작 템플릿 생성

![Untitled](/assets/img/Infra/AWS/private/Untitled%208.png)

EC2탭에서 대상 인스턴스 이미지화를 합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%209.png)

이미지를 생서합니다. ‘재부팅 안함’ 옵션은 체크해줍니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2010.png)

EC2- AMI 탭에가면 만들어진 AMI가 보입니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2011.png)

EC2 - 시작 템플릿 탭에서 Auto Scaling의 기준이 되는 템플릿을 만듭니다. 여기서는 Auto Scaling 지침 옵션을 체크합니다. 그래야 아래에서 AMI 를 선택하는 것이 필수가 됩니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2012.png)

내 AMI 탭으로 가서 아까 만든 AMI를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2013.png)

네트워크는 나중에 설정해야해서 - 포함하지 않음을 선택합니다.

### ☑️Auto Scaling 그룹 생성

![Untitled](/assets/img/Infra/AWS/private/Untitled%2014.png)

EC2 - Auto Scaling에서 Auto Scaling 그룹 생성을 하면 시작템플릿을 골라야합니다. 아까 만든 템플릿을 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2015.png)

네트워크 매핑에서 중요한건 pvt 만들었다고 pvt 선택을 하는것이 아니라 , 퍼블릭 서브넷을 선택해야 합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2016.png)

로드 밸런서는 ALB로 합니다. Internet-facing = 인터넷 경계

![Untitled](/assets/img/Infra/AWS/private/Untitled%2017.png)

리스너에서는 대상그룹이 필요한데 새로 만들겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2018.png)

상태확인을 체크합니다. 나중에 CloudWatch로 확인할거거든요!

![Untitled](/assets/img/Infra/AWS/private/Untitled%2019.png)

그룹 크기를 정합니다. 최소2 최대4로 하겠습니다. 크기 조정 정책은 나중에 세팅하겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2020.png)

알림 설정을 합니다 옛날에 만든 SNS 주제를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2021.png)

태그 설정하여 Auto Scaling할 때 생성되는 인스턴스 이름 asg로 통일합니다.

### ☑️Route 53 설정

![Untitled](/assets/img/Infra/AWS/private/Untitled%2022.png)

Route 53 탭에서 ALB 레코드 생성을 합니다.

### ☑️HTTPS 설정

![Untitled](/assets/img/Infra/AWS/private/Untitled%2023.png)

EC2 - 로드 밸런서 - 만들어진 ALB를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2024.png)

리스너에 https 추가를 하고 Auto Scaling할 때 생성했던 대상그룹으로 전달을 합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2025.png)

그리고 ACM에서 요청했던 인증서를 선택합니다.

### ☑️Auto Scale Out 단순 크기 조정 정책 설정

![Untitled](/assets/img/Infra/AWS/private/Untitled%2026.png)

Auto Scaling 그룹에서 만들었던 my-asg로 이동합니다. 아직은 정책이 없는게 보입니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2027.png)

동적 크기 조정 정책 생성합니다. 먼저 확장하는 정책(Scale out)부터 하겠습니다. 유형은 단순 크기로 하겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2028.png)

CloudWatch 경보생성 눌러서 새로운 생성 페이지로 갑니다. 지표선택 누르고 EC2 - Auto Scaling 그룹별 - CPUUtilization을 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2029.png)

화면처럼 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2030.png)

그러면 현재 cpu그래프가 보입니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2031.png)

조건탭에서 cpu 사용률이 70%가 넘어가면 경보를 울리게 합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2032.png)

알람이 울렸을 때 작동할 작업 트리거를 설정하겠습니다. 만들어 놓은 my-sns주제를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2033.png)

![Untitled](/assets/img/Infra/AWS/private/Untitled%2034.png)

여기까지 하면 아까 CloudWatch 생성이 되는겁니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2035.png)

그럼 다시 동적 크기 조정으로 돌아와서 아까 생성한 cloudwatch 경보를 선택합니다. 그리고 작업 수행은 ‘추가’로 하고 1개로 합니다. 이러면 cloudwatch가 scaleout 알람을 울렸을 때 인스턴스를 하나 추가하게 됩니다.

### ☑️Auto Scale In 단계 크기 조정 정책 설정

![Untitled](/assets/img/Infra/AWS/private/Untitled%2036.png)

이제는 scale in을 해보겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2037.png)

아까랑 똑같이 cloudwatch 경보를 생성합니다. 마찬가지로 cpu util쪽으로 가서 지표를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2038.png)

조건은 30%보다 작을때로 하겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2039.png)

마찬가지로 my-sns 주제를 선택합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2040.png)

이 경보의 이름을 정합니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2041.png)

그런 다음 이 경보가 울렸을 때의 작업 트리거를 설정합니다. cpu가 30보다 작아졌을 때 인스턴스를 하나 제거하기로 하겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2042.png)

이제 auto scaling그룹에 my-asg그룹에는 동적 크기 조정 정책이 2개가 있는것을 확인할 수 있습니다.

### ☑️트래픽 부하 주기

![Untitled](/assets/img/Infra/AWS/private/Untitled%2043.png)

auto scaling 그룹 최소 ec2 용량 개수가 2라서 2개가 있습니다. 각각 구분할 수 있도록 asg01, asg02로 하겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2044.png)

각 인스턴스마다 `**yes > /dev/null &`\*\* 명령어를 입력하여 인스턴스 에 최대 부하를 주겠습니다.

![Untitled](/assets/img/Infra/AWS/private/Untitled%2045.png)

![Untitled](/assets/img/Infra/AWS/private/Untitled%2046.png)

이제 아까 만든 scaleout 경보를 가면 임계치를 넘어갔고 따라서 새로운 인스턴스가 생성된 것을 확인 할 수 있습니다.
