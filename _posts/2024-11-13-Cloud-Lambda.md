---
title: "[Cloud] Lambda, Eventbridge의 이해"
date: 2024-11-13 00:00:00 +0900
categories: [클라우드 인프라 구축 프로젝트]
tags: [cloud, aws, lambda, eventbridge, cloudwatch]
render_with_liquid: false
---

# AWS Lambda & EventBridge

## 1. AWS Lambda

개발 안내서 : [https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/welcome.html](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/welcome.html)

### ServerLess 란?

- 서버 관리와 프로비저닝 없이 코드 실행을 가능하게 하는 클라우드 컴퓨팅 모델.
- 서버리스(Server-less)는 서버가 없는 백엔드 라는 뜻이 아닌 우리가 직접 서버를 관리하지 않아도 된다는 의미
    - 서버리스 아키텍처(Serverless Architecture)란 서버를 직접 관리할 필요가 없는 아키텍처.
    - 사이드 프로젝트나 빠르게 프로토타입을 출시할 때 빠르고 쉽게 제품을 출시할 수 있고, 비용도 매우 절약할 수 있다.
- 특징
    - 서버 관리 부담 제거
    - 자동 스케일링 제공
    - 사용한 만큼만 비용 지불
- 이점
    - 인프라 관리의 간소화
    - 빠른 개발 주기와 비용 절감 가능성

### AWS Lambda 란?

- AWS Lambd는 서버를 프로비저닝하거나 관리하지 않고도 **애플리케이션 서버나 백엔드 서비스에 대한 코드를 실행**할 수 있는 **이벤트 중심의 서버리스 컴퓨팅 서비스**이다.
- AWS Lambd는  특정 이벤트(예: HTTP 요청, 데이터베이스 업데이트, 파일 업로드 등)가 발생할 때 트리거되어 함수를 실행.
- AWS Lambd를 사용하면 Lambda가 지원하는 언어 런타임 중 하나로 코드를 제공하면 된다.
    - 다양한 지원 언어 : Python, Node.js, Java, Go, C#, Ruby 등 여러 언어 지원.
- AWS Lambd는 다양한 AWS  서비스와 연동되어 사용

### **AWS Lambda를 써야하는 이유**

- **서버 관리 불필요**: 인프라 관리가 필요 없으며, 오직 코드와 이벤트에 집중 가능.
- **자동 확장**: 사용자 요청에 따라 자동으로 인스턴스를 늘리거나 줄임.
- **비용 효율적**: 실제 코드가 실행된 시간만큼 비용을 지불하여, 미사용 시 비용 발생하지 않음.
- **통합성**: 다양한 AWS 서비스와 쉽게 통합 가능 (S3, DynamoDB, API Gateway 등).

### **AWS Lambda의 특징 및 제한**

- 람다는 **함수 코드**와 코드를 실행하기 위한 **런타임**으로 구성되어 있음.
- **함수 실행 시간 제한**: **최대 15분 실행** 시간 제한.
- **코드 크기 제한**
    - 코드 패키지는 최대 50MB(zip 파일)
    - 전체 디플로이 크기는 최대 250MB(unzip 파일)
    - 콘솔에디터 사용 시 : 3MB
- **메모리 및 리소스**: 메모리 128MB 에서 10240MB(10GB) 사이 설정 가능.
- **권한 관리**: IAM을 사용해 권한 부여 필요.

### **AWS Lambda 요금제**

- **요금 구조**: [https://aws.amazon.com/ko/lambda/pricing/](https://aws.amazon.com/ko/lambda/pricing/)
    - **요청 횟수** : 월 1백만 건의 요청까지 무료 제공, 이후 요청 당 요금 발생.
    - **실행 시간** : 1ms 단위로 측정하며, 실행 메모리 크기와 실행 시간에 따라 비용 부과.
    - **프리 티어** : 월 1백만 개 요청과 40만GB-초 실행 시간 무료 제공.
- **비용 절감 팁**: 함수를 최적화하여 실행 시간을 줄이거나, 필요한 리소스를 최소화하여 비용을 줄일 수 있음.

### **AWS Lambda 서비스 적용 사례**

- **웹 애플리케이션 백엔드**: API Gateway와 연계해 API 백엔드를 서버리스로 구축.
- **데이터 처리 파이프라인**: S3에 업로드 된 파일을 실시간으로 분석하여 Lambda로 처리.
- **IoT 데이터 분석**: IoT 센서 데이터 수집 및 필터링, 데이터베이스에 저장.
- **자동화된 알림 시스템**: 클라우드 서비스의 상태 변화를 감지하고, 자동으로 이메일이나 Slack 알림 전송.
- **클라우드 기반 서비스 배치 작업**: CloudWatch Events를 사용해 정기적인 데이터 백업이나 상태 점검 스크립트 실행.

## 2. EventBridge

### **EventBridge란?**

- AWS 서비스 및 자체 애플리케이션에서 발생하는 **이벤트를 모니터링**하고, **이를 기반으로 여러 AWS 서비스나 외부 SaaS 응용 프로그램과의 통합을 자동화하는 이벤트 버스 서비**스.
- 실시간으로 다양한 소스의 이벤트를 수신하고 이를 라우팅하여 특정 작업을 트리거하는 역할을 수행
- **특징**:
    - **서버리스**: 인프라 관리가 필요 없는 완전 관리형 서비스.
    - **이벤트 기반 아키텍처** 지원: 실시간 데이터 통합 및 반응형 애플리케이션 구축 가능.

### **EventBridge의 주요 구성 요소**

- **이벤트 소스**: AWS 서비스, SaaS 애플리케이션, 맞춤형 애플리케이션에서 생성되는 이벤트.
- **이벤트 버스**: 이벤트를 수집하고 규칙에 따라 라우팅하는 경로로, 기본 AWS 이벤트 버스와 사용자 정의 이벤트 버스를 사용할 수 있음.
- **규칙(Rule)**: 특정 이벤트 조건을 지정하여 특정 조건에 맞는 이벤트만 타겟으로 라우팅.
- **타겟(Target)**: 이벤트가 전송되는 대상으로, Lambda 함수, Step Functions, SQS, SNS 등 다양한 AWS 서비스와 통합 가능.

### **EventBridge 요금제**

- **요금 구조**:
    - **이벤트 인입**: 인입되는 이벤트마다 비용이 발생하며, 월별 무료 티어 제공.
    - **스키마 레지스트리 및 디스커버리**: 스키마 탐색과 저장에 대한 비용 발생.

## 3. LAB : AWS Lambda & EventBridge를 통한 EC2 인스턴스 시작 및 중지

### 1분마다 인스턴스 stop 되도록 구성하기

### 3.1. 정책 만들기

- AWS Lambda가 CloudWatch Logs에 로그를 기록하고, 모든 EC2 인스턴스를 중지할 수 있도록 허용하는 정책을 생성
- IAM > [정책] > [정책생성] > 정책편집기-[JSON] 클릭 실행후 아래의 JSON 기반 정책을 복사 붙이기 후 [다음]
    - JSON code
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "logs:CreateLogGroup",
                        "logs:CreateLogStream",
                        "logs:PutLogEvents"
                    ],
                    "Resource": "arn:aws:logs:*:*:*",
                    "Effect": "Allow"
                },
                {
                    "Action": [
                        "ec2:Stop*"
                    ],
                    "Resource": "*",
                    "Effect": "Allow"
                }
            ]
        }
        ```
        
        - 의미
            - "Version": "2012-10-17": 이 필드는 정책 언어의 버전을 지정. 2008년 버전, 2012년 버전 등 AWS는 여러 정책 언어 버전을 지원하며, 2012-10-17은 현재 가장 일반적으로 사용되는 버전임.
            - "Statement"는 정책의 주된 요소로, 여기에는 하나 이상의 권한 부여 명령이 포함됨
                - 첫번째 statement
                "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"]
                    - Lambda 함수가 CloudWatch Logs에 로그를 기록할 수 있도록 허용합니다.
                    CreateLogGroup: 로그 그룹을 생성할 수 있는 권한.
                    CreateLogStream: 로그 스트림을 생성할 수 있는 권한.
                    PutLogEvents: 로그 이벤트를 로그 스트림에 기록할 수 있는 권한.
                    
                    "Resource": "arn:aws:logs:*:*:”
                    *어떤 리소스에 대해 액세스가 허용되는지를 지정. arn:aws:logs:*:*:*는 모든 리전의 모든 로그 그룹과 스트림에 대한 접근을 허용함.
                    
                    "Effect": "Allow": 이 작업들을 허용한다는 의미.
                    
                - 두 번째 Statement:
                "Action": ["ec2:Stop*"]: 이 부분은 Lambda 함수가 EC2 인스턴스를 중지할 수 있는 권한을 허용. Stop*는 StopInstances 등과 같은 모든 EC2 중지 관련 작업을 포함합니다.
                "Resource": "*": 이 작업이 모든 리소스에 대해 허용된다는 의미. 즉, 계정 내 모든 EC2 인스턴스를 대상으로 할 수 있다는 의미.
                "Effect": "Allow": 이 작업들을 허용한다는 의미.

- 1단계: 권한지정
    - 정책 이름 :  **AllowStopInstance**
    - 정책에 정의된 권한 : 서비스 CloudWatch Logs, EC2
    - [정책 생성]
- 생성된 정책 내용 확인

### 3.2. IAM 역할 생성

- 생성된 정책을 Lambda 에서 사용하기 위해서는 IAM 롤을 생성해야 함.
- IAM > [역할] - [역할 생성]
    - 1단계: 신뢰할 수 있는 엔터티 유형  : 람다를 신뢰할수 있는 엔터티로 설정해야한다.
        - AWS 서비스
        - 사용 사례 : Lambda ****선택 후 [다음]
            
            ![image.png](/assets/img/Cloud/lambda/image.png)
            
    - 2단계: 권한추가
        - 롤이 사용할 정책을 정의
        - AllowStopEc2 선택 후 AWSLamda를 입력해서
            - **AllowStopInstance**
            - AWSLambdaBasicExecutionRole
            - AWSLambdaVPCAccessExecutionRole
            - 선택 후 [다음]
    - 3단계: 이름 지정, 검토 및 생성
        - 역할이름 : **myStopinatorRole**
            
            ![image.png](/assets/img/Cloud/lambda/image%201.png)
            
        
        [역할 생성]
        

### 3.3.  Lambda 함수 생성

- AWS Lambda > [함수 생성]
    - [새로 작성] 선택 후
    - 기본 정보
        - 함수 이름 : **myStopinatorRoleFunc**
        - 런타임 언어 : Python 3.12 최신버전 선택
        - 아키텍처 : x86_64
    - 권한
        - 기본 실행 역할 변경
            - 기본 역할 사용  선택 후 **myStopinatorRole** 선택
    
    [함수 생성]
    

### 3.4. 함수가 호출될 수 있는 규칙 생성

- [트리거 추가]
    - 트리거 구성 : EventBridge(CloudWatch Events) 선택
        
        **EventBridge는 과거 CloudWatch Events**라 불렀고,
        
        AWS 서비스, 타사 애플리케이션, 또는 사용자 정의 애플리케이션에서 발생하는 **이벤트를 감지하고**, **이를 사전에 정의된 규칙에 따라 다른 서비스로 라우팅**합니다. 
        예를 들어, EC2 인스턴스가 시작될 때 발생하는 이벤트를 감지하여 특정 Lambda 함수를 호출할 수 있습니다.
        여기에서는 특정 시간 또는 특정 이벤트 발생 시 앞서 3에서 만든 Lambda함수  myStopinatorRoleFunc를 호출합니다.
        
    - 규칙 : EventBridge가 감지할 이벤트를 기록 - 여기에서는 1분마다 한번씩 이벤트를 일으켜 에서 만든 Lambda함수  **myStopinatorRoleFunc**를 호출하려고 한다.
        - 새 규칙 생성
            - 규칙이름 : **everyMinute**
            - 규칙설명: **everyMinute**
            - 규칙 유형 : 예약 표현식 - Cron(반복적)식이나 rate(일회성) 표현식을 사용할 수 있다
                - 예약 표현식 : **rate(1 minute)** 입력 후 [추가]
                - 참고
                    - cron 예제
                        
                        
                        | **예** | **Details** |
                        | --- | --- |
                        | cron(0/30 * * * ? *) | 30분마다 |
                        | cron(0 0/1 * * ? *) | 매 시간 |
                        | cron(0 0/2 * * ? *) | 2시간마다 |
                        | cron(0 0/4 * * ? *) | 4시간마다 |
                        | cron(0 0/8 * * ? *) | 8시간마다 |
                        | cron(0 0/12 * * ? *) | 12시간마다 |
                        | cron(15 13 ? * * *) | 매일 오후 1시 15분 |
                        | cron(15 13 ? * MON *) | 매주 월요일 오후 1시 15분 |
                        | cron(30 23 ? * TUE#3 *) | 매월 세 번째 화요일 오후 11:30 |
                    - rate 예제
                        
                        
                        | **예** | **Details** |
                        | --- | --- |
                        | rate(30 minutes) | 30분마다 |
                        | rate(1 hour) | 매 시간 |
                        | rate(5 hours) | 5시간마다 |
                        | rate(15 days) | 15일마다 |
                    - 참고: [https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/with-eventbridge-scheduler.html](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/with-eventbridge-scheduler.html)
                    - 참고: [https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/reference-cron-and-rate-expressions.html](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/reference-cron-and-rate-expressions.html)
                    - 참고 : [https://docs.aws.amazon.com/ko_kr/scheduler/latest/UserGuide/schedule-types.html#cron-based](https://docs.aws.amazon.com/ko_kr/scheduler/latest/UserGuide/schedule-types.html#cron-based)
                    - crontab 이해를 위한 링크 : [https://crontab.guru/](https://crontab.guru/)
            - 생성
                
                ![image.png](/assets/img/Cloud/lambda/image%202.png)
                

### 3.5. 람다 함수에서 실행할 코드 설정

lambda > myStopinstorRoleFunc의 [코드] 탭에서 기존의 내용 삭제하고 아래 Python 코드 기록

- 기존코드 : 람다함수 호출되면 200. Hello from Lambda!를 송출한다.
    
    ```json
    import json
    
    def lambda_handler(event, context):
        # TODO implement
        return {
            'statusCode': 200,
            'body': json.dumps('Hello from Lambda!')
        }
    
    ```
    
- 새로 변경할 코드
    - Lambda 함수는 지정된 EC2 인스턴스를 중지하고, 중지된 인스턴스 ID를 출력
    - <REPLACE_WITH_REGION>와 <REPLACE_WITH_INSTANCE_ID>는 사용 중인 리전과 인스턴스_ID로 변경합니다.
    
    ```python
    import boto3
    region = '<REPLACE_WITH_REGION>'
    instances = ['<REPLACE_WITH_INSTANCE_ID>']
    ec2 = boto3.client('ec2', region_name=region)
    
    def lambda_handler(event, context):
        ec2.stop_instances(InstanceIds=instances)
        print('Stopped your instances:' + str(instances))
    ```
    

변경 후 적용 **[Deploy]** → 1분 기준으로 인스턴스 중지되는지 확인하고 로그 확인

### 3.6. CloudWatch에 lambda가 로그 그룹, 로그 스트림을 생성하고 로그를 기록했는지 확인

- CloudWatch > 로그 그룹 에서 **/aws/lambda/myStopinatorRoleFunc 확인 →** 로그 스트림 생성되었는지 확인(예: 2024/09/23/[$LATEST]c3…943f9) 클릭해서 생성된 로그 확인
    
    ![image.png](/assets/img/Cloud/lambda/image%203.png)
    

---

## 4. LAB2 : AWS Lambda & EventBridge 설정 정보 수정 적용

### **4.1 3**분마다 인스턴스 stop 되도록 구성하기

- EventBridge에서 이벤트 발생 시간을 3분으로 조절하면 된다.
- Lambda > **myStopinatorRoleFunc > [구성] → EventBridge everyMinute** 선택해서 [이벤트 일정] [편집]을 눌러 3분으로 수정 - 다음 - 다음.. [규칙 업데이트]

### 4.2 **1**분마다 인스턴스를 **start** 하도록 구성하기

1.  EventBridge 규칙 업데이트
    - Lambda > **myStopinatorRoleFunc > [구성] → EventBridge everyMinute** 선택해서 [이벤트 일정] [편집]을 눌러 1분으로 수정 - 다음 - 다음.. [규칙 업데이트]

2. 정책 수정

- AWS Lambda가 CloudWatch Logs에 로그를 기록하고, 모든 EC2 인스턴스를 시작할 수 있도록 허용하는 정책을 추가
- IAM > [정책] > [AllowStopStartInstance] 생성
    - JSON code
        
        참고 : [https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/APIReference/Welcome.html](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/APIReference/Welcome.html) > Actions
        
        ```json
        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "logs:CreateLogGroup",
                        "logs:CreateLogStream",
                        "logs:PutLogEvents"
                    ],
                    "Resource": "arn:aws:logs:*:*:*",
                    "Effect": "Allow"
                },
                {
                    "Action": [
                        "ec2:Stop*",
                        "ec2:Start*"
                    ],
                    "Resource": "*",
                    "Effect": "Allow"
                }
            ]
        }
        ```
        
- Role - ec2-start-Role

3. Lambda 함수 코드 수정

- lambda > ec2-start-RoleFunc 생성
    
    ```python
    import boto3
    region = '**<REPLACE_WITH_REGION>**'
    instances = ['**<REPLACE_WITH_INSTANCE_ID>**']
    ec2 = boto3.client('ec2', region_name=region)
    
    def lambda_handler(event, context):
        ec2.start_instances(InstanceIds=instances)
        print('Started your instances:' + str(instances))
    ```
    
    - 참고 : [https://boto3.amazonaws.com/v1/documentation/api/latest/index.html](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)
        - Available Services → EC2

4. 이벤트 발생을 종료 시키기

lambda →  구성 → everyMinute → 비활성화

리소스 삭제

- EventBrige 삭제
- IAM
    - 역할 삭제
    - 정책 삭제
- Lambda 삭제
- CloudWatch - 로그 그룹 삭제
- EC2삭제

과제: Lambda를 사용하여 매일 저녁 6시(한국시간)에 정기적으로 **모든 Amazon EC2 인스턴스를 중지**하도록 구성

- **예약 기반(Schedule-based ) 람다 기능 예제: EC2 인스턴스 시작 및 중지**
    
    ![image.png](/assets/img/Cloud/lambda/image%204.png)
    

- AWS Lambda 및 Amazon CloudWatch 이벤트를 구성하여 미리 정의된 시간에 EC2인스턴스를 Start 또는  Stop하도록 구성
    - CloudWatch의 이벤트를 통해 Lambda 함수를 실행하여 EC2 인스턴스를 시작 또는 종료할 예정
    - 진행 단계
        - 람다 함수는 EC2 인스턴스를 중지할 수 있는 권한을 부여하는 IAM 역할로 트리거되고 실행됩니다.
        - CloudWatch를 통해 특정 시간이 되면 이벤트 발생
        - Lambda는 IAM 역할을 통해 EC2를 시작/종료 할 수 있는 권한을 할당 받아 인스턴스를 종료/시작을 실행

- 참조링크
    - **Lambda를 사용해서 Amazon EC2 인스턴스를 정기적으로 중지 및 시작하려면 어떻게 해야 하나요?**
        
        참고 : [https://repost.aws/ko/knowledge-center/start-stop-lambda-eventbridge](https://repost.aws/ko/knowledge-center/start-stop-lambda-eventbridge)
        
    - 특정 태그가 있는 실행 중인 EC2 인스턴스를 자동으로 종료하는 기능을 제공
        - [https://gist.github.com/mlapida/1917b5db84b76b1d1d55](https://gist.github.com/mlapida/1917b5db84b76b1d1d55)