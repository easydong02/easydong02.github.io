---
title: 데이터타입과 서식지정자
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

데이터타입?

전에 출력편에서 서식지정자를 잠깐 보았는데요 오늘은 그 서식 지정자와 데이터타입에 대해서 배운 것을 써보려고 합니다.

C언어의 데이터 타입은

char: 1byte, 문자하나(정수)

short : 2byte, 정수

int: 4byte, 정수

long: 4byte, 정수(시스템마다 다름)

float: 4byte, 실수

double: 8byte, 실수

로 되어 있습니다. 여기서 1byte는 8bit(2진법에서 8자릿수)로 용량을 나타내는 데요, 앞서 변수를 선언할 때 쓰였던 int도 저 중 하나입니다.

그렇다면 서식지정자는,

printf의 포맷문자열 안에 들어가는 서식지정자(format specifier)들입니다.

%d: 10진수 정수로 출력

%c: 문자로 출력

%f: 실수로 출력

%x: 16진수로 정수로 출력

%%: %출력

이제 이것을 활용해보도록 하겠습니다.

![Desktop View](/assets/img/Programming-Language/C/DataType-Format/1.png){: width="100%" }

변수 Number와 ch1을 각각 정수와 문자로 하나씩 출력해보겠습니다. 여기서 문자는 홀따옴표( '')를 씁니다.

![Desktop View](/assets/img/Programming-Language/C/DataType-Format/2.png){: width="100%" }

10a 붙어서 나오니 가독성이 없네요 다음부턴 \\n를 넣어서 줄바꿈을 하도록 하겠습니다.

**정수타입**

정수타입, 용량의 차이는

1byte / 8bit =2의 8승 = 256개(가지)

0~255까지 담을 수 있는 용량

char: -128 ~ +127

short: -32768 ~ +32767

int: -2147483648 ~ +2147483647

입니다. 따라서 각각에 필요한 곳에 필요한 타입과 용도를 알고 쓰면 더 좋겠죠?

그렇다면 char타입으로 변수를 선언하고 초기값으로 128를 넣고 %d로 정수를 출력한다면 어떻게 출력이 될까요?

놀랍게도 -128이 출력됩니다! 그 이유는 오버플로우(overflow)에서 찾아볼 수 있는데요.

char는 정수를 나타낼때 -128부터 +127까지 되는데요. 마치 시계처럼 한 바퀴가 돌면 다시 처음으로 갑니다. 따라서 127 다음엔 제일 처음인 -128로 돌아가는 것이죠. 같은 원리로 129를 넣는다면 출력은 -127로 되겠네요!

![Desktop View](/assets/img/Programming-Language/C/DataType-Format/3.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/DataType-Format/4.png){: width="100%" }

**실수타입**

실수타입은 float과 double이 있는데요 각각 4byte, 8byte의 용량을 가지고 있습니다.

각각의 용량의 차이는 정밀도의 차이인데요, float은 소숫점 6~9자리 double은 소숫점 15~18자리까지 표현이 가능합니다.

출력하고 싶은 소숫점 자리는 서식지정자를 쓸 때 .x로 x번째 소숫점 자리까지만 표현하게 할 수 있습니다.

따라서 %.2f는 2번째 소숫점까지 표현한 실수타입의 서식지정자를 일컫습니다!

이번 시간은 여기까지 하도록 하겠습니다!
