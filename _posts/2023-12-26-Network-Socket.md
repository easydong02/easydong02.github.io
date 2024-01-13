---
title: "[Network] 네트워크 소켓"
date: 2023-12-26 00:00:00 +0900
categories: [Network, Study]
tags: [network, url, message]
render_with_liquid: false
---

## 네트워크 소켓

### 프로토콜 스택의 내부 구성

- OS에서는 브라우저에서 받은 메시지를 서버에 송출하는 동작에서 2가지가 있다.

  - 네트워크 제어용 소프트웨어: 프로토콜 스택
  - 네트워크 하드웨어: LAN 어댑터

- 프로토콜 스택 내부 구성

| APP - 네트워크 어플리케이션(브라우저, 메일러, 웹 서버, 메일 서버 등) <br/>Socket 라이브러리(리졸버 포함)                                                                                                                                                                                                  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OS - 프로토콜 스택(TCP + UDP / IP) <br/> TCP: 브라우저나 메일 등의 일반적인 어플이 데이터를 송/수신할 경우에 사용 <br/> UDP: DNS 서버에 대한 조회 등에서 짧은 제어용 데이터를 송/수신할 경우에는 UDP<br/> IP: 패킷을 통신 상대까지 운반하는 역할과 ICMP와 ARP라는 프로토콜을 다루눈 부분도 포함하고 있다. |
| 드라이버 SW - LAN드라이버(어댑터 제외)                                                                                                                                                                                                                                                                    |
| HW - LAN 어댑터                                                                                                                                                                                                                                                                                           |

### 소켓의 실체는 통신 제어용 제어 정보

- 소켓이란 개념적인 것이어서 어떠한 실체를 가진 것은 아니다.
- 한편 프로토콜 스택 내부에 제어 정보를 기록하는 메모리 영역이 있고, 여기에 IP주소, 포트 번호, 진행 상태 등이 존재한다.
- 소켓은 이 제어 정보라고 할 수 있다. 따라서 응답의 여부와, 요청을 보낸 경과 시간 등으로 제어를 할 수도 있다.
- 즉, 프로토콜 스택은 소켓의 제어 정보를 따라 움직인다.

![Desktop View](/assets/img/Network/Study/Socket/1.png)

- 여기서 보이는 netstat의 명령어 결과의 하나하나의 행이 소켓이다.

### Socket을 호출했을 때의 동작

1. 소켓을 만든다. 따라서 생성된 소켓이 들어갈 자리(메모리)를 확보한다.
2. 제어 정보(소켓)을 메모리에 할당한다. 그리고 생성된 소켓을 식별한 디스크립터 고지.
3. 그리고 애플리케이션은 프로토콜 스택에 데이터 송/수신을 의뢰할 때 이 열쇠(디스크립터)만 가지고 호출한다. 이 열쇠(디스트립터)로 지정한 상자(소켓)을 열면 거기 안에 제어 정보가 담겨있기 때문이다.

## Story02 ㅡ 서버에 접속한다.

### 접속의 의미

- socket()을 호출하면 소켓이 생성되는데 클라이언트와 서버가 LAN선 같은 물리적인 연결 상태가 아니기 때문에 처음 생성한 소켓에는 아무것도 기록되어 있지 않다.
- 따라서 클라이언트 측 소켓에는 보낼 서버의 IP주소와 포트번호 등을 기재해야 하고
- 서버측 소켓에는 클라이언트로부터 송신 측의 IP와 포트번호를 받게 된다.
- 데이터 송/수신에는 이동되는 데이터를 일시적으로 저장하는 메모리 영역이 필요한데, 이를 '버퍼 메모리'하고 부른다.

### 맨 앞부분에 제어 정보를 기록한 헤더를 배치한다.

- 데이터를 송/수신 할 때는 데이터를 '패킷' 단위로 잘라서 보낸다.
- 이 패킷을 보낼 때 '헤더'를 같이 동행시켜서 보낸다.
- 이 헤더가 제어 정보이다. 이더넷이나 IP의 제어 정보, TCP의 제어 정보 등이 포함된다.
- 여기에는 송/수신 측의 포트 번호나 컨트롤 비트나 추가적으로 올 수 있는 옵션 등이 있다.
- socket(), connect(), write(), read(), close() 각 단계에서 대화할 때마다 이 제어정보가 부가된다.
- 제어 정보의 종류
  - 헤더에 기입되는 정보
  - 소켓(프로토콜 스택의 메모리 영역)에 기록되는 정보

### 접속 동작의 실체

- connect()를 호출하면서 서버측의 IP 주소와 포트 번호를 넘기면 TCP부분에 전달됨.
- 중요한 것은 송/수신처의 포트 번호이다. 이거로 상대측의 소켓을 지정할 수 있기 때문.
- 현재는 헤더에 있는 컨트롤 비트인 SYN을 1로 만들어서 접속한다고 보면 된다.
- 그렇게 되면 TCP부분에서 IP부분으로 넘어가면서 본격적으로 송/수신이 진행된다.
- (클라이언트) TCP - IP - IP - TCP (서버)
- 서버측에서 받은 데이터를 다시 클라이언트로 송신할 때 ACK컨트롤 비트를 1로 만든다. 이것은 패킷을 받을 것을 알리기 위함.
- 그리고 최종적으로 클라이언트에서 받았다고 끝이 아니라 잘 받았다는 TCP헤더를 반송하면 접속이 끝이 난다.

## Story03 ㅡ 데이터를 송/수신한다.

### 프로토콜 스택에 HTTP 리퀘스트 메시지를 넘긴다.

- connect()에서 app에 제어가 다시 돌아오면 본격적으로 송/수신 작업에 돌입한다.
- 프로토콜 스택은 받은 데이터를 곧바로 송신하진 않는다. 버퍼 메모리에 단위 크기만큼 저장을 하고 보낸다.
- 네트워크 효율을 위해서 어느정도 데이터를 저장하고 송신하는데 그 척도가 MTU이다.
- MTU = MSS(패킷에서 데이터의 최대 길이) + 헤더
- 두번째 척도는 타이밍이다. 송신 속도가 느릴 때는 다 모을때까지 기다리는 것보다 적절히 송신을 진행시키는 것이 좋다.

### 데이터가 클 때는 분할하여 보낸다.

- 한 개의 패킷보다 큰 데이터를 송신할 때는 TCP/IP가 조각으로 분리하여 각 조각에 헤더를 붙여서 송신한다.

### ACK번호를 사용하여 패킷이 도착했는지 확인한다.

- 송신을 하면 데이터를 보내고 끝이 아니라 잘 갔는지 확인도 필요하다.
- 그래서 시퀀스 번호(패킷의 처음 데이터 순번)를 TCP헤더에 기록한다.
- 패킷에서 헤더를 빼면 데이터가 남으니까 이 길이를 이용한다.
- 하나의 패킷의 데이터 길이가 N 이라면, 시퀀스번호+N이 ACK 번호가 된다.
- 따라서 ACK번호와 다음 패킷의 시퀀스 번호가 일치하면 데이터 누락이 없는 것이다.
- ACK번호 = 시퀀스 번호 + 데이터 길이, 송신한 데이터에 대응하는 ACK번호가 오지 않는다면 데이터가 중간에 누락된 것.

### 패킷 평균 왕복 시간으로 ACK 번호의 대기 시간을 조정한다.

- 타임아웃 값: ACK 번호가 돌아오는 것을 기다리는 시간.
- 이 타임아웃의 시간을 넘으면 다시 패킷을 보낸다(회복 처리)
- 이 타임아웃 값은 네트워크 상황에 따라서 동적으로 정해진다.
- ACK 번호가 돌아오는 시간을 평균을 내서 정한다.

### 윈도우 제어 방식으로 효율적으로 ACK 번호를 관리한다.

- 클라이언트에서 패킷을 송신하고 손가락 빨지는 않는다. 계속 보낸다.
- 서버측에선 수신 버퍼를 설정하여 오는 데이터를 처리하고 다시 보내준다.
- 수신 버퍼에서 나가는 것보다 쌓이는 속도가 더 크면 들어오는 데이터가 지낼 공간이 줄어든다.

### ACK 번호와 윈도우를 합승한다.

- 윈도우 통지의 타이밍: 수신 측의 수신 버퍼의 빈 영역이 발생했을 때 통지하는 것.
- 수신측은 ACK 윈도우 통지를 바로 보내지 않고 다음 통지 동작이 일어나면 ACK 번호와 윈도우 통지를 한 개의 패킷으로 만든다.

### HTTP 응답 메시지를 수신한다.

- HTTP 리퀘스트 메시지를 보내면 거기에 맞는 응답 메시지를 read()요청한다.
- 그러면 다시 제어가 프로토콜 스택에 넘어간다.
- 그리고 수신 버퍼에서 수신 데이터를 추출하여 App에 건네줌.
- 이때 리퀘스트 메시지의 송신이 얼마 되지 않았다면 아직 응답이 안 왔을 가능성이 크다.
- 이럴 땐 받은 데이터가 없으므로 App에게 건네주는 작업을 보류함.

## Story04 ㅡ 서버에서 연결을 끊어 소켓을 말소한다.

### 데이터 보내기를 완료했을 때 연결을 끊는다

- 데이터 보내기를 완료한 쪽에서 연결 끊기 도입
- 서버측이 끊는다고 하면 close()호출
- 컨트롤 비트의 FIN 비트 1로 변경 IP에 클라이언트측으로 송신 의뢰.
- 클라이언트는 ACK번호 서버로 반송
- 어플리케이션이 데이터 받아오면 연결 종료

### 소켓을 말소한다

- 소켓을 바로 말소하진 않는다. FIN이 도착할 때까지 살아있어야 한다.
- 소켓이 말소되면 그 포트번호로 새로운 소켓이 생성될 수 있고 따라서 데이터 수신이 되지 않기 때문.
- 몇 분 정도 기다리고 나서 소켓 말소.

### 데이터 송/수신 동작을 정리한다.

## Story05 ㅡ IP와 이더넷의 패킷 송/수신 동작

### 패킷의 기본

- 패킷의 기본형: 헤더 + 데이터(내용)
- TCP/IP의 패킷: (MAC헤더 + IP헤더 + (TCP헤더 + 데이터 조각))
- 송신처에서 헤더를 부착한 뒤 헤더를 따라서 중계기 간 이동이 이루어짐
- IP헤더는 최종 목적지가 들어있음
- MAC헤더는 다음 라우터 목적지가 들어있음

### 패킷 송/수신 동작의 개요

- IP는 송출만한다. TCP와 달리 데이터가 누락됐는지 등은 신경쓰지 않는다.
- 운반은 허브나 라우터와 같은 네트워크 기기가 전담
- LAN 어댑터에 전해진 것은 전기나 빛의 신호이고 0과 1의 데이터가 아니다.

### 수신처 IP 주소를 기록한 IP 헤더를 만든다.

- IP헤더는 TCP 헤더 앞에 붙는다.
- 들어가는 내용은 송/수신처 IP 주소, 프로토콜 번호 등
- 패킷을 운반시킬 상대를 판단하는 방법은 경료표 이용.
- 목적지와 넷마스크 0.0.0.0이면 기본 게이트웨이를 뜻함
- 게이트웨이는 다음 라우터의 IP주소 기록, 인터페이스는 LAN어댑터와 같은 네트워크용 인터페이스

### 이더넷용 MAC 헤더를 만든다.

- 수신처 MAC주소: 전달하는 상대의 MAC주소
- 송신처 MAC주소: 이 패킷을 송신한 측의 MAC 주소
- 이더 타입: 사용하는 프로토콜의 종류

### ARP로 수신처 라우터의 MAC 주소를 조사한다.

- ARP: Address Resolution Protocol
- IP주소를 MAC주소로 반환

## Story06 ㅡ UDP 프로토콜을 이용한 송/수신 동작

### 수정 송신이 필요없는 데이터의 송신은 UDP가 효율적이다

- TCP는 데이터를 확실하게, 효율적으로 보내야 한다.
- 따라서 어떤 패킷이 누락됐는지의 회복 과정을 위해서 과정이 복잡하다
- 이 때, 데이터를 '전부'보내고 수신측의 확인을 요청하면 간단하다.
- 수틀리면 그냥 다시 전부 보내버리면 되기 때문.

### 제어용 짧은 데이터

- DNS 서버에 대한 정보 조회는 패킷 하나에서 끝난다.
- 따라서 복잡하게 TCP 프로토콜을 사용할 이유가 없다.
- UDP를 쓰면 송/수신이 간단해진다.

### 음성 및 동영상 데이터

- 음성이나 영상 데이터는 결정된 시간 안에 데이터를 건네주어야 한다.
- 따라서 중간의 패킷의 누락에서 그 부분만 보내는 것이 아니라 데이터 특성상 '전부' 보내야 한다.