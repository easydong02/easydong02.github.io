---
title: "[AWS] Auto Scaling"
date: 2024-05-09 00:00:00 +0900
categories: [Infra, AWS]
tags: [aws, autoscaling]
render_with_liquid: false
---

## ✅ Auto Scaling이란?

---

애플리케이션의 로드를 처리할 수 있는 **정확한 수의 인스턴스를 보유하도록 보장**할 수 있습니다. **Auto Scaling 그룹**이라는 인스턴스 모음을 생성합니다. 각 Auto Scaling 그룹의 최소 인스턴스 수를 지정할 수 있으며, Auto Scaling에서는 그룹의 크기가 이 값 아래로 내려가지 않습니다. 각 Auto Scaling 그룹의 최대 인스턴스 수를 지정할 수 있으며, Auto Scaling에서는 그룹의 크기가 이 값을 넘지 않습니다. **원하는 용량을 지정한 경우 그룹을 생성한 다음에는 언제든지 Auto Scaling에서 해당 그룹에서 이만큼의 인스턴스를 보유할 수 있습니다.**

## ✅ Auto Scaling 그룹 생성

---

### ☑️ 기존 인스턴스 이미지로 만들기

![Untitled](/assets/img/Infra/AWS/auto/Untitled.png)

먼저 Auto Scaling을 하려면 기준이 되는 인스턴스 이미지가 필요합니다. 그래서 원래 있던 web서버 인스턴스를 이미지로 만듭니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%201.png)

보시면 스냅샷이 먼저 만들어지고 대기중으로 되어있습니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%202.png)

사용가능 상태가 되면 AMI에서도 만든 이미지를 볼 수 있습니다. 다음 단계로 진행합니다.

### ☑️ 시작 템플릿 생성

![Untitled](/assets/img/Infra/AWS/auto/Untitled%203.png)

이제 스케일링할 대상 템플릿을 생성합니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%204.png)

OS 이미지를 선택하면 아까만든 AMI 가 보입니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%205.png)

키페어는 원래 있던 키페어로하고 보안그룹은 기존에 만들었던 web서버 보안그룹을 선택합니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%206.png)

생성을하면 시작템플릿 목록에 방금 만든 템플릿이 존재합니다.

### ☑️ 오토 스케일링 그룹 생성

![Untitled](/assets/img/Infra/AWS/auto/Untitled%207.png)

이제 오토 스케일링을 실제로 할 그룹을 생성합니다. 아까 만든 시작 템플릿을 선택합니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%208.png)

vpc는 제가 만든 vpc와 서브넷도 마찬가지로 합니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%209.png)

로드 밸런서는 새 로드밸런서로 하고 ALB를 선택합니다. 그리고 체계는 Internet-facing을 선택합니다. 이게 번역이 한글로 나왔다 영어로 나왔다 하니 Internet-facing단어를 잘 기억합시다!

![Untitled](/assets/img/Infra/AWS/auto/Untitled%2010.png)

리스너 및 라우팅에선 대상 그룹을 생성해봅시다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%2011.png)

원하는 용량(그룹 크기)는 초기 크기입니다 2개로 할경우 아까만든 AMI가 2개로 인스턴스로 되어 시작합니다. 그리고 크기 조정에서는 최소 용량과 최대 용량을 설정합니다. 대상 추적 크기 조정 정책을 선택하면 관련 지표에 따라 동적으로 크기가 조정됩니다. 저는 정책 없음으로 수동으로 해보겠습니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%2012.png)

유지 관리 정책에서는 인스턴스 교체 이벤트의 동작 방식을 설정합니다. 저는 만약 인스턴스가 교체될 때는 새로운 인스턴스가 시작될때까지는 다른 인스턴스가 살아있게 하겠습니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%2013.png)

이메일로 알림을 받겠습니다.

![Untitled](/assets/img/Infra/AWS/auto/Untitled%2014.png)

태그에는 asg로 Name 키값을 정해줍니다!
