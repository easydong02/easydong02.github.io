---
title: "[AWS] CloudFront로 정적 웹 호스팅"
date: 2024-08-13 00:00:00 +0900
categories: [Infra, AWS]
tags: [aws, cdn, cloudfront, s3]
render_with_liquid: false
---

## ✅CloudFront?

---

.html, .css, .js 및 이미지 파일과 같은 정적 및 동적 웹 콘텐츠를 사용자에게 더 빨리 배포하도록 지원하는 웹서비스입니다. cloudfront는 엣지 로케이션이라고 하는 데이터 센터의 전 세계 네트워크를 통해 콘텐츠를 제공합니다. CloudFront를 통해 서비스하는 콘텐츠를 사용자가 요청하면 지연 시간이 가장 낮은 엣지 로케이션으로 라우팅되므로 콘텐츠 전송 성능이 뛰어납니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled.png)

## ✅ 실습

---

### ☑️S3 남미 버킷 생성

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%201.png)

남미에 S3 버킷 만들겠습니다. 그 이유는 S3에 접속할 때 cloudFront의 효과를 체감하기 위해서입니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%202.png)

버킷을 생성합니다. 구분하기 쉽게 도메인 앞 단어는 paulo라고 했습니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%203.png)

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%204.png)

퍼블릭 액세스를 열어줍니다!

### ☑️폴더 및 파일 업로드

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%205.png)

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%206.png)

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%207.png)

버킷 생성확인을 하고 간단한 html파일을 업로드 하고 퍼블릭 액세스 권한도 따로 열어줍니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%208.png)

버킷 안에 이미지 파일을 담을 폴더를 생성합니다. 실제 폴더 로직은 아닌데 구분자(”/”)를 사용하여 폴더처럼 사용합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%209.png)

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2010.png)

생성된 폴더에 이미지를 추가합니다. 당연히 퍼블릭 액세스도 설정합니다.

### ☑️S3 정적 웹 호스팅 설정

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2011.png)

버킷 - 속성 - 스크롤 내리고 정적 웹 사이트 호스팅 편집에 들어갑니다. 비활성화가 되어있네요.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2012.png)

웹 호스팅 편집에서 활성화 해주시고 인덱스 문서는 아까 추가한 index.html 파일로 설정합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2013.png)

Route53 - 호스팅 영역에서 레코드 생성 해준 뒤, 아까 버킷의 이름을 입력합니다. 그리고 라우팅 대상은 생성한 버킷!

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2014.png)

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2015.png)

페이지에 들어가지는데 응답속도가 668ms. 페이지 잘 나오긴 하는데 응답 시간이 느리죠? 이제 cloundFront를 사용 해봅시다.

### ☑️다른 리전에서 ACM 생성

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2016.png)

ACM을 생성합니다. 여기서 중요한 점은, **cloundFront ACM은 반드시 버지니아 북부 리전에서만 사용이 가능하여 그 리전에서 생성해야 합니다.** 도메인 이름은 우리의 도메인으로 합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2017.png)

생성된 인증서를 확인합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2018.png)

그리고 `Route 53에서 레코드 생성`  버튼을 눌러서 사용하고 있는 도메인을 선택하여 레코드를 생성합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2019.png)

Route 53 호스팅 영역에  CNAME 레코드 생성 확인합니다.

### ☑️CloudFront 배포 생성

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2020.png)

ColudFront 탭으로 가서 생성 - 기존에 만들었던 S3 버킷 선택합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2021.png)

기본 설정으로 합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2022.png)

나머지는 다 똑같은데 ‘뷰어 프로토콜 정책’만 ‘HTTPS only’로 합니다. 왜냐하면 ACM 인증서를 사용하여 접속하기 때문입니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2023.png)

엣지 로케이션의 캐시 정책, 디폴트 설정으로 합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2024.png)

이것도 마찬가지로 디폴트 설정

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2025.png)

WAF 설정은 하지 않습니다. 과금이 되기 때문입니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2026.png)

설정에서 cname 설정으로 대체 도메인 이름을 설정합니다, 중요한 건 ACM은 버지니아 북부 리전에서만 생성된 것을 선택할 수 있다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2027.png)

나머지 설정들도 디폴트로 합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2028.png)

생성하고 ‘마지막 수정’ 부분이 ‘배포중’이 아니라 최근 시각이 나온다면 배포 된 것입니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2029.png)

Route 53 - 레코드 생성 - 생성한 CloudFront 엔드포인트 설정합니다.

![Untitled](/assets/img/Infra/AWS/cloudfront/Untitled%2030.png)

아까와 같은 버킷에 접속을 하는데 77ms로 엄청 빨라졌습니다!