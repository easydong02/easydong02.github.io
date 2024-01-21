---
title: "[Network] OSI 7계층"
date: 2024-01-22 00:00:00 +0900
categories: [Network, Study]
tags: [network, osi]
render_with_liquid: false
---

# **네트워크 - OSI 7계층: 계층별로 알아보는 네트워크의 동작**

안녕하세요! 오늘은 네트워크에서 사용되는 OSI 7계층에 대해 자세히 알아보겠습니다. OSI 모델은 Open Systems Interconnection의 약자로, 네트워크 통신의 각 단계를 7개의 계층으로 나눈 모델입니다. 각 계층은 특정한 역할을 수행하며, 계층별로 독립적으로 동작하여 통신의 효율성과 안정성을 증가시킵니다.

## **1. 물리 계층 (Physical Layer)**

물리 계층은 데이터를 전기적이거나 광학적인 신호로 변환하고, 전송 매체를 통해 실제로 전송하는 역할을 합니다. 전기적 신호로 변환된 데이터는 케이블, 허브 등을 통해 이동하게 됩니다. 대표적인 장비로는 허브, 리피터 등이 있습니다.

### 📌프로토콜

- **Ethernet (이더넷):** 유선 네트워크에서 가장 흔하게 사용되는 LAN 프로토콜. 데이터 프레임을 물리적인 신호로 변환합니다.
- **USB (Universal Serial Bus):** 주로 컴퓨터와 주변 장치 간의 연결에 사용되는 표준 인터페이스. 물리적인 연결과 데이터 전송을 담당합니다.

## **2. 데이터 링크 계층 (Data Link Layer)**

데이터 링크 계층은 물리 계층을 통해 송수신되는 데이터의 오류와 흐름을 관리합니다. 이 계층에서는 프레임(Frame)이라는 단위로 데이터를 관리하며, 주소 값(MAC 주소)을 통해 특정 장치에게 데이터를 전달합니다. 대표적인 장비로는 스위치, 브릿지 등이 있습니다.

### 📌프로토콜

- **MAC (Media Access Control):** 네트워크 카드에 할당된 고유한 식별자. 프레임의 출발지와 목적지를 식별합니다.
- **ARP (Address Resolution Protocol):** IP 주소를 MAC 주소로 매핑하는 프로토콜. IP 주소로 통신하기 위해 해당하는 MAC 주소를 찾습니다.
- **PPP (Point-to-Point Protocol):** 점대점 연결을 제공하는 프로토콜. 주로 모뎀과 라우터 사이의 통신에 사용됩니다.

## **3. 네트워크 계층 (Network Layer)**

네트워크 계층은 여러 개의 네트워크를 거칠 때마다 경로를 찾아주는 역할을 합니다. 이를 라우팅(Routing)이라고 하며, IP 주소를 기반으로 데이터를 목적지까지 전달합니다. 라우터가 이 계층에서 동작하며, IP 프로토콜을 사용합니다.

### 📌프로토콜

- **IP (Internet Protocol):** 인터넷에서 데이터의 전달을 담당하는 프로토콜. IP 주소를 사용하여 목적지를 식별하고 경로를 선택합니다.
- **ICMP (Internet Control Message Protocol):** 네트워크에서 메시지를 전송하고 에러를 알리기 위한 프로토콜. 주로 'ping'과 'traceroute' 등에 사용됩니다.
- **OSPF (Open Shortest Path First):** 라우팅을 위한 프로토콜. 네트워크에서 최적의 경로를 찾아주는 역할을 합니다.

## **4. 전송 계층 (Transport Layer)**

전송 계층은 종단 간(End-to-End) 통신을 제공하며, 신뢰성 있는 데이터 전송을 담당합니다. 데이터의 분할 및 조립, 흐름 제어, 오류 복구 등의 기능을 수행합니다. 대표적인 프로토콜로는 TCP(Transmission Control Protocol)와 UDP(User Datagram Protocol)가 있습니다.

### 📌프로토콜

- **TCP (Transmission Control Protocol):** 신뢰성 있는 연결 기반의 프로토콜. 데이터를 세그먼트로 분할하여 전송하고, 수신 측에서 재조립합니다.
- **UDP (User Datagram Protocol):** 신뢰성이 낮지만 빠른 데이터 전송을 제공하는 프로토콜. 실시간 응용 프로그램에 적합합니다.

## **5. 세션 계층 (Session Layer)**

세션 계층은 양 끝단의 사용자 간에 통신을 관리하고 연결을 설정, 유지, 종료하는 역할을 합니다. 세션을 만들고 유지하는 동안 데이터 교환을 조정하며, 데이터의 동기화 및 복구를 담당합니다.

### 📌프로토콜

- **NetBIOS (Network Basic Input/Output System):** LAN에서 통신을 하기 위한 프로토콜. 주로 Windows 환경에서 사용됩니다.
- **RPC (Remote Procedure Call):** 원격 프로시저 호출을 통해 분산 환경에서 프로세스 간 통신을 지원하는 프로토콜.

## **6. 표현 계층 (Presentation Layer)**

표현 계층은 데이터를 응용 계층에서 이해할 수 있도록 변환하거나, 응용 계층에서 받은 데이터를 네트워크를 통해 전송할 수 있도록 변환하는 역할을 합니다. 데이터의 압축, 암호화, 인코딩 등의 작업이 이루어집니다.

### 📌프로토콜

- **JPEG (Joint Photographic Experts Group):** 이미지 압축을 위한 표준 프로토콜. 디지털 이미지의 압축과 해제에 사용됩니다.
- **SSL/TLS (Secure Sockets Layer/Transport Layer Security):** 보안 통신을 위한 프로토콜. 웹에서의 안전한 데이터 전송을 지원합니다.

## **7. 응용 계층 (Application Layer)**

응용 계층은 최종 사용자의 인터페이스를 제공하며, 사용자의 요청에 따라 서비스를 수행합니다. 이 계층에는 여러 응용 프로그램과 서비스가 포함되며, HTTP, FTP, SMTP 등의 프로토콜이 이용됩니다.

### 📌프로토콜

- **HTTP (Hypertext Transfer Protocol):** 웹 서버와 웹 브라우저 간의 통신을 위한 프로토콜. HTML 문서 전송에 사용됩니다.
- **FTP (File Transfer Protocol):** 파일 전송을 위한 프로토콜. 서버와 클라이언트 간 파일 전송을 지원합니다.
- **DNS (Domain Name System):** 도메인 이름과 IP 주소를 매핑하는 시스템. 도메인 이름을 IP 주소로 변환합니다.
