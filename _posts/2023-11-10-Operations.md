---
title: 연산자
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

연산자에는 여러가지가 있다. 산술연산자, 복합 대입 연산자, 증감 연산자, 논리 연산자, 삼항 연산자, 비트 연산자 등..

하나씩 알아보자.

**산술 연산자(+,-,\*,/, %)**

덧셈, 뺄셈, 나눗셈, 곱셈은 알겠는데 %이것은 뭘까? 바로 나머지 연산자이다.

예를 들어, 5%2 라는 것은 5를 2로 나눈 나머지, 즉 1인 셈이다.

![Desktop View](/assets/img/Programming-Language/C/Operations/1.png){: width="100%" }

이렇게 문장을 작성하고 출력을 하면

![Desktop View](/assets/img/Programming-Language/C/Operations/2.png){: width="100%" }

이런식으로 1이 출력된다!

**복합 대입 연산자(+=, -=, \*=, /=, %)**

대부분의 이항연산자에 대한 복합대입연산자 가능하다

예를 들어 int n = 10 에서,

n = n + 3 을 하게 되면 기존의 변수 값에 +3 증가가 된다. 그것을 간단하게 표현 하자면,

n += 3 이렇게 된다. 즉 자기 자신에게 3을 대입하고 그것을 중첩시킨 것이다.

![Desktop View](/assets/img/Programming-Language/C/Operations/3.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/4.png){: width="100%" }

n은 0에서 3을 중첩시켜서 출력이 잘 되었다.

**증감 연산자(++,--)**

증감 연산자는 ++,--를 쓰면 되고 각각 1씩 증가, 1씩 감소를 듯한다. 예를 들어, n++ 면 기존의 n값에 1을 하나 더하는 것이다.

![Desktop View](/assets/img/Programming-Language/C/Operations/5.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/6.png){: width="100%" }

이런식으로 1이 증가된 것을 확인할 수 있다.

주의 할 점은 n++와 ++n이 다르다는 점인데 이것은 다항연산식에서 순서의 차이가 있다.

n++: postfix 방식 : 기존연산 '직후' 에 증감연산 수행

++n: prefix 방식 : 기존연산 '직전' 에 증감연산 수행

하지만 가급적이면 증감연산자는 연산식 '안'에서 사용하는 것은 피하는게 좋다.

**비교연산자( > , < , >=, <=, ==, != ...)**

결과가 '거짓'이면 0 '참'이면 보통1, 하지만 0이 아닌 모든 숫자는 '참'이다.

![Desktop View](/assets/img/Programming-Language/C/Operations/7.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/8.png){: width="100%" }

이런식으로 정수서식지정자에 논리를 넣었더니 그 값이 1과 0으로 구성되는 것을 알 수 있다.

**논리연산자(&&, ||, !)**

논리 연산자는 어떤 단순한 숫자의 계산이 아닌 논리의 참과 거짓을 가르는 연산자이다.

&& : and 연산자, 논리곱 연산, 피연산자 양쪽이 둘다 '참' 인 경우만 참

|| : or 연산자, 논리합 연산 , 피연산자 양쪽이 둘중 하나만 참이어도 참

! : not 연산자, 참->거짓, 거짓->참

예를 들어,

T && T == > T(1)

T && F == > F(0)

F && T == > F(0)

F && F == > F(0)

T || T == > T(1)

T || F == > T(1)

F || T == > T(1)

F || F == > F(0)

!T == > F(0)

!F == > T(1)

이런 식으로 결과값이 나온다.

![Desktop View](/assets/img/Programming-Language/C/Operations/9.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/10.png){: width="100%" }

이렇게 result 값이 그 논리 연산자의 참과 거짓으로 나뉘어 1과 0으로 출력됨을 알 수 있다.

NOT 연산자( !)

만약 10==10이라는 논리는 참이고 그 값은 1이 될 것이다. 하지만 여기에 !(10==10)을 하게 되면 그 반대로, 즉 0값이 출력된다.

![Desktop View](/assets/img/Programming-Language/C/Operations/11.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/12.png){: width="100%" }

이렇게 말이다.

**삼항 연산자**

( 조건식 ) ? (참인경우 결과) : (거짓인 경우 결과)의 형식으로 쓰여진다.

조건식을 정해 놓고 참인 경우의 결과값과 거짓인 경우 결과값을 출력하는데 거짓인 경우에도 ?의 기호를 붙여서 또 다른 삼항 연산자를 이어 갈 수 있다.

![Desktop View](/assets/img/Programming-Language/C/Operations/13.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/14.png){: width="100%" }

밑의 식 같은 경우 또 다른 삼항연산자를 이어 붙여봤다.

**비트 연산자**

비트 연산자는 논리와 다르게 그 값의 bit단위로 계산하는 식이다. 장점이라면 컴퓨터는 이 비트를 움직이고 비교하는 것이 우리가 알고 있는 산술 연산보다 훨씬 더 빠르게 계산한다.

& : AND 연산자

| : OR 연산자

^ : XOR 연산자

~ : NOT 연산자

<<, >> : SHIFT 연산자

**& 연산자**는 2진법 수로 봤을때 두 수에서 각각의 자릿수가 둘 다 1일때만 1로 출력한다. 따라서 &연산자를 사용하면 그 값이 대체로 더 작아진 값이 나온다.

![Desktop View](/assets/img/Programming-Language/C/Operations/15.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/16.png){: width="100%" }

신기하다!

**| 연산자**는 두 이진법 나열 중에서 각각의 자릿수에서 두 수중 하나라도 1이 있으면 1을 출력한다. 따라서 | 연산자를 사용 하면 대체로 값이 커진다.

![Desktop View](/assets/img/Programming-Language/C/Operations/17.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/18.png){: width="100%" }

값이 제대로 예상되로 출력 되었다.

**^ 연산자: XOR 비트 연산자(eXclusive OR : 배타적 논리합)**

^연산자는 둘다 같으면 0, 다르면 1로 출력한다. 헷갈리니 바로 예시로 가자

![Desktop View](/assets/img/Programming-Language/C/Operations/19.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/20.png){: width="100%" }

재밌다!

**~ 연산자 : NOT 비트 연산자**

간단하게 말하자면 비트 반전이다 1<->0

![Desktop View](/assets/img/Programming-Language/C/Operations/21.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/22.png){: width="100%" }

잉? 갑자기 왜 ~120이 -121로 되었을까? 이것은 용량을 봐야한다.

십진법 120은 2진법으로 111 1000이다. 하지만 4byte타입(32bit)으로 보자면 이것은

0000 0000 0000 0000 0000 0000 0111 1000 (120)이다 여기서 비트 반전을 줘버려서

1111 1111 1111 1111 1111 1111 1000 0111 (~121)이 된것이다. 이것은 큰 숫자라서 전에 얘기했던

오버플로우(overflow)가 되어서 음수가 된 것이다.

**Shift 연산자( <<, >>)**

bit 단위로 이동하는 연산자이다.

num1 << 2; : 이것은 비트단위 왼쪽으로 2자리 이동(shift) 하는 것이다

![Desktop View](/assets/img/Programming-Language/C/Operations/23.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/24.png){: width="100%" }

이렇게 된다!

이번시간에는 연산자에 대해서 좀 길게 써봤다~

