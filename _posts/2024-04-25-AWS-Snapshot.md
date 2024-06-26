---
title: "[AWS] 스냅샷(Snapshot)"
date: 2024-04-25 00:00:00 +0900
categories: [Infra, AWS]
tags: [aws, ebs, snapshot, volume]
render_with_liquid: false
---

## ✅ EBS란?

---

Amazon Route53을 도메인 DNS서비스 (예: example.com)로 사용할 수 있습니다. Route53이 DNS 서비스인 경우, www.example.com과 같은 친숙한 도메인 이름을 컴퓨터 간 연결에 사용되는 192.0.2.1 등 숫자IP주소로 변환하여 인터넷 트래픽을 웹사이트로 라우팅합니다. 사용자가 브라우저에 도메인 이름을 입력하거나 이메일을 보내면 DNS 쿼리가 Route53에 전달되며이에 따라 적절한 값으로 응답합니다. 예를 들어 Route53은 example.com 웹서버의 IP주소를 사용하여 응답할 수 있습니다.

## ✅ 볼륨 생성

---

![Untitled](/assets/img/Infra/AWS/snapshot/Untitled.png)

EC2 - 볼륨 탭으로 갑니다 저는 web01 인스턴스에 붙을 볼륨을 하나 추가하려고 합니다. 볼륨생성 버튼을 누릅니다.

![Untitled](/assets/img/Infra/AWS/snapshot/Untitled%201.png)

이렇게 gp2에 1GB 볼륨이 만들어졌습니다. 이거를 이름을 바꿔서 web01-add로 합니다. 중요한건 원래의 web01-root와 가용영역이 같아야 합니다.

![Untitled](/assets/img/Infra/AWS/snapshot/Untitled%202.png)

이렇게 잘나오는게 보이구요 이거를 인스턴스에 볼륨 연결을 해봅시다 . 작업 - 볼륨연결누릅니다.

![Untitled](/assets/img/Infra/AWS/snapshot/Untitled%203.png)

원하는 인스턴스를고르고 sdb로 파티션하여 연결합니다!

```bash
lsblk

sudo mkfs -t xfs /dev/xvdb 파일시스템

sudo mount /dev/xvdb /mnt 마운트

sudo umount /mnt 언마운트
```

주의할 점은 디스크 조회할 때 `lsblk`는 마운트와 상관없이 볼륨 연결이 되면 조회가 되고 `df -h`는 마운트를 해야 조회가 된다!
