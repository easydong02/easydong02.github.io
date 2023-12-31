---
title: "[Java] 변수(Variables) 연산자(Operation)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

드디어 3개월 간의 선수과정이 끝나고 국비로 들어왔습니다. 저녁에 3시간씩 하던 때와는 다르게 하루에 9시부터 저녁 6시까지 하려니까 확실히 선수과정보다 길고 나가는 진도도 어마어마합니다. 하지만 하나하나 이뤄가면서 저의 꿈에 더 가까워지는 그런 사다리 역할을 한다고 생각하니까 감회가 새롭습니다. JAVA는 선수과정으로 2달정도 들었지만 항상 초심과 겸손으로 처음배우는 것처럼 더 다듬어 가도록 하겠습니다!

**1\. 프로그래밍 언어**

**1-1. 프로그래밍 언어란?**

\- 프로그래밍 언어란 주어진 어떤 문제를 해결하기 위해 인간과 컴퓨터 사이에서 의사소통을 가능케 하는 인공적인 언어입니다.

\- 이 언어를 통하여 사용자는 컴퓨터에게 일련의 일을 시키는 명령어들의 집합체인 프로그램을 작성할 수 있습니다.

**1-2. 프로그래밍 언어의 종류**

\- 기계어 : 컴퓨터가 이해하는 언어로서 2진수의 집합으로 구성되어 있습니다. 이것은 사람이 당연히 이해할 수 없겠죠?그래서 언어툴이 생겨났습니다.

\- 고급언어 : 사람이 이해할 수 있는 수준의 언어, 기계어로 변환되어야만 프로그램 형태로 실행하는 것이 가능합니다. 이것을 우리가 사용하는거죠. (C, C++,JAVA)

**1-3. JAVA**

\- 썬 마이크로시스템즈에서 개발하여 1996년 1월에 공식적으로 발표한 객체지향 프로그래밍 언어.

\- 운영체제에 독립적

\-> JVM이 설치된 환경이라면 어디서든지 실행 가능합니다.

\- 객체지향 언어

\-> 상속, 캡슐화, 다형성

\-> 코드의 재사용 및 유지보수에 용이합니다.

\- 자동 메모리 관리

\-> Garbage Collector가 자동으로 메모리를 관리해줍니다.

\- 네크워크, 분산처리, 멀티스레드

\-> 시스템과 관계없이 네트워크, 분산처리, 멀티스레드 구현을 위한 손쉬운 API제공

**1-4. Java 프로그램이 만들어지는 과정**

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/1.png)

**.java 파일은 혼자서 구동될 수 없다. 이것은 class파일로 컴파일러가 변환해주며 이 클래스 파일들을 JVM이 운영체제와 소통한다.**

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/2.png)

**컴파일러가 변환해준 class파일을 JVM이 각 운영체제에 맞게 바이트 코드를 전달한다.**

**1-5. Java 가상머신(Java Virtual Machine/JVM)**

\- 컴파일된 자바 바이트 코드를 실행시켜 주는 소프트웨어

\- 자바 프로그램은 JVM 설치된 환경이라면 운영체제나 하드웨어 종속되지 않고 실행이 가능합니다.

\- 운영체제로부터 독립적입니다.

\- One Source - Multi Use

\- 각 운영체제에 맞는 JVM이 설치된 환경이라면, 하나의 프로그램이 실행 환경의

영향을 받지 않고 동일하게 실행될 수 있습니다.

**2\. 주석문**

**2-1. 주석문이란?**

\- 프로그램 소스코드 안에 개발자의 필요에 따라 명시하는 설명문

\- 주석문은 프로그램으로 컴파일되지 않습니다.

\- 특정 명령문이 실행되지 않도록 차단하는 용도로 사용할 수 있습니다.

**2-2. 주석문의 종류**

\- 한 줄만 처리하는 주석문

// "//"가 앞에 명시된 라인은 주석으로 인식됩니다.

\- 여러 줄을 처리하는 주석문

/\*

이 블록 안에서는 여러 라인을 주석으로 처리할 수 있습니다.

\*/

해봅시다.

```
package HelloJava;

/*
 * 프로그램 소스의 최소 단위(=class)를 명시하는 블록.
 * -> public class 클래스 이름
 * 클래스 이름은 소스파일의 이름과 동일해야 하며, 영어/숫자/언더바(_)만 사용가능하다.
 * 첫 글자는 반드시 영어로만 구성되어야 한다.
 */

public class HelloJava {
	
	// 프로그램의 시작점을 의미하는 블록 -> main 메서드라고 한다.
	public static void main(String[] args) {

		// 콘솔에 문장을 출력하기 위한 명령어
		// 문장을 표현하는 방법 -> 쌍따옴표로 묶는다 -> 문자열
		System.out.println("Hello Java");
		
	}

}
```

보시면 HelloJava라는 클래스를 만들었습니다. 주석이 /\*\*/형의 주석과 //형의 주석 모두 쓰였습니다. 여기서 제가 느낀 것은

main메소드였는데요, 그냥 무심코 지나쳤는데 이것이 프로그램의 시작점을 의미하는 블록이었습니다. 정확히 알고보니 반갑네요 ^^

그리고 println()출력메소드가 사용되었습니다. 문자열 "Hello Java"가 출력이 되었네요.

**3\. 변수의 이해**

**3-1. 변수의 이해**

f(x) = x + 1

\-Java에서도 다양한 종류의 자료를 표혀날 수 있는 값을 변수라고 부르며, 일반적으로 웹 페이지에서

처리하고자 하는 "데이터"에 해당합니다.

**3-2. 자료형(data type)**

\- 프로그래밍 언어에서 변수의 종류를 구별하기 위해 사용되는 키워드

\- 자바에서 제공되는 자료형의 종류에는 총 8가지가 있으며, 이를 기본 자료형(Primitive Data Type)이라 합니다.

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/3.png)

**3-3. 자료형이 가지는 의미**

\- 모든 변수는 컴퓨터의 메모리 영역에 생성됩니다. 하나의 저장공간이죠.

\- 일반적으로 메모리란, PC에 설치하는 RAM을 의미합니다.

\- 4byte의 크기를 갖는 변수값 하나를 생성할 경우, PC의 RAM에서 해당 용량만큼을 사용하게 됩니다.

\- 변수는 RAM의 크기를 넘지 않는 범위 안에서만 생성할 수 있습니다.

(용량을 넘어서는 경우 OutOfMemory에러 발생) 이 경우 혼자 작업할때는 볼 일이 없지만, 실무에서 다같이 공유하는 프로그램일 경우 메모리 사용량을 초과할 때도 있습니다.

**3-4. 문자열 데이터**

\- 프로그램 코드에서 "문장"을 표현하기 위해 사용되는 데이터 값.

\- String입니다. 이 부분으 아시다시피 정말 어려운 파트니 나중에 다시 다뤄보도록 하겠습니다.

\- 문자열을 표현하기 위한 자료형이다. (첫 글자 대문자 주의)

\- 글자 수에 상관 없이 쌍따옴표("")로 묶인 내용을 할당할 수 있습니다.

\- 숫자값의 경우 쌍따옴표로 묶이게 되면 문자열로 취급되므로, 숫자와 문자열을 분명하게 구별해야 합니다.

**3-5. 변수의 사용 방법**

\- 변수를 사용하는 방법은 "선언"과 "할당" 의 두 영역으로 구분된다.

\- 변수의 선언

\-> 선언은 데이터 형과 사용하고자 하는 변수의 이름을 지정한 후,

세미콜론(;)으로 한 라인을 종료한다.

데이터형 변수이름;

\- 변수의 할당

\-> 선언된 변수에 원하는 값을 대입하는 과정을 의미한다.

값의 대입은 대입연산자("=")를 사용하며, 우변에서 좌변으로 대입된다.

변수이름 = 값;

\- 사용 예

```
		int num1;	//변수의 선언
		num1 = 100;	//변수의 할당
```

\-선언과 할당의 통합(선언과 할당은 다음과 같이 한 줄로 표현될 수 있다.)

데이터형 변수이름 = 값;

\- 사용 예

```
int num2 = 1000;
```

\- 문자열 데이터

\-> String 형의 변수를 선언하고, 상 따옴표로 묶인 값을 대입한다.

\-> 빈 문자열이나 공백도 문자열 데이터이다.

```
String msg = "안녕하세요. 자바 "; //문자를 표현 (공백 포함)

String black = ""; //빈 문자열

String age = "22"; //문자열(비록 숫자값이 있다하더라도 쌍 따옴표 안에 있음
```

\- 문자열 데이터의 덧셈

\-> 문자열 + 문자열 : 두 문장을 하나로 합쳐준다.

```
String language = "JA" + "VA"; //JAVA 출력
```

\-> 문자열 + 기본자료형 : 기본 자료형의 데이터가 문자열로 변환되고, 두 문장이 합쳐진다.

```
int age = 20;

String name = "자바학생";

String result = name + age; //"자바학생" + "20" -> "자바학생20"
```

**3-6. 변수이름의 명명 규칙**

\- 변수 이름은 영문, 숫자, "\_", "$"만 사용 가능합니다.

\- 변수 이름의 첫 글자는 숫자로 시작될 수 없습니다.

\- 대/소문자를 엄격하게 구별하므로 오타에 주의해야 합니다. (name != Name)

\- 자바에서 사용하는 예약어(키워드)를 사용할 수 없습니다.

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/4.png)

**JAVA 예약어**

ex)myName userPassword user\_input

\- 클래스 이름의 명명 규칙도 변수 이름의 규칙과 동일합니다.

**3-7. 값의 할당 방법**

\-boolean은 true(참) false(거짓)중의 한 가지만 갖습니다.

```
boolean isKorean = true;

boolean isJapanese = false;
```

\-char는 홑따옴표(')로 감싸진 한 글자만 대입할 수 있다.

```
char first = '곽';
```

\-long, float, double은 다른 데이터형과의 구별을 위해 값 뒤에 데이터 형의 첫 글자를 접미사로 가질 수 있다.

접미사는 대/소문자를 가리지 않지만 가급적 대문자로 사용하는 것이 좋다.

```

long money = 5000000L;

float pi = 3.14F;

double lat = 128.32452D;

```

\- 생성된 변수는 다른 변수에 대입될 수 있다.

```

int num1 = 100;

int num2 = num1;

```

\- 변수 사용의 제약

\-> 반복하여 다른 값이 할당될 수 있지만, 선언은 중복 불가

```

int num1 = 200;

System.out.println(num1); // 200

num1 = 500;

System.out.println(num1); // 500

int num1 =300; //이미 선언된 변수이므로 에러

```

\-> 선언되지 않은 변수는 사용할 수 없다.

```

int num1 = 200;

num1 = 500;

num2 = 700; //선언되지 않은 변수이므로 에러

```

\-> 값이 대입되지 않은 변수는 변수에 대입하거나 출력할 수 없다.

```

int num1;

int num2 = num1; //할당되지 않은 변수를 대입하였으므로 에러

System.out.println(num1); //할당되지 않은 변수를 출력하였으므로 에러

```

**3-8. 상수 = 변하지 않는 수**

\- 변수와 마찬가지로 메모리상에 존재는 하지만, 값이 변경될 수 없는 데이터

\- final 키워드를 사용하여 선언된 변수는 상수로 생성된다.

```

final int age = 20;

final long money = 1200000L;

final float PI = 3.14F; //상수 변수 이름은 대문자

```

\-메모리상에 이름이 생성되므로 접근은 가능하지만, 할당한 값을 변경할 수는 없다.(읽기 전용)

```

final int age = 22;

age = 23; //상수의 값을 변경하므로 에러

```

**4\. 연산자**

\- 프로그램에서 연산을 수행하기 위하여 사용되는 특수기호들

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/5.png)

**4-1. 대입연산자**

\- 대입 연산자(=)는 좌변에 우변을 대입한다는 의미입니다.

\- 수학에서는 "="기호가 대입한다는 의미와 같다는 의미로 함께 사용되지만, java에서의 "="기호는 "대입한다"는 의미로만 사용됩니다.

\- 변수에 값을 대입하는 경우 - 일반적인 변수의 할당 과정

```

double PI = 3.14D;

int monet = 12000;

```

\-변수에 변수를 대입하는 경우

```

int a = 3;


int b =a; //b에 a의 값이 복사된다.

```

**4-2. 사칙(산술)연산자**

\-일반적인 덧셈(+), 뺄셈(-), 곱셈(\*), 나눗셈(/,%)을 수행합니다.

```
		int num1 =12;
		int num2 =8;
		int result1 = num1 + num2;
		int result2 = num1-num2;
		int result3 = num1 * num2;
		int result4 = num1 / num2;
		int result5 = num1 % num2;
```

\-사칙연산의 결과는 대입연산자를 통하여 다른 변수에 대입될 수 있습니다.

```
int result6 = result1;
```

\-정수(byte,short,int,long)와 실수(float,double)의 연산시에는 정수가 실수 형태로 자동 변환되어 처리되기 때문에

결과는 실수가 됩니다.

```
double a = 3.14;
int b = 3;
double c = a+b;
```

\- 나눗셈에 있어서의 주의사항

\-> 10나누기 3을 계산 할 때, 수학에서는 몫3 나머지 1이라고 계산되지만,

java에서는 나눗셈에 대해 두 개의 연산자로 구분됩니다.

\-> 10/3 : 나눗셈의 몫만을 취하여 결과값은 "3"입니다.

\-> 10%3 : 나눗셈의 나머지만을 취하여 결과값은 "1"입니다.

\- 프로그램에서의 연산은 2진수로 변환되어 이루어집니다.

실수는 2진수로의 변환이 되지 않기 때문에, 실수의 나눗셈은 오차가 발생합니다.

\- 모든 수는 0으로 나눌 수 없다.

\-연산자 우선순위

\-> 곱셈(\*)과 나눗셈(/,%)은 덧셈(+)과 뺄셈(-)보다 우선합니다.

\-> 여러 연산자를 복합적으로 사용할 경우, 괄호로 묶여 있는 곳이 우선합니다.

**4-3. 단항 연산자**

\- 어떤 변수(x)의 값에 대한 계산 결과를 다시 자기 자신에게 대입하고자 하는 경우의 약식 표현입니다.

```

int x =100;

x =x +5;

---------단항 연산자 적용---------


int x= 100;

x +=5;

```

\-단항 연산자는 모든 사칙 연산에 표현 가능합니다.

\-> +=, -=, \*=, /=, %=

**4-4. 증감 연산자**

\- 단항 연산자로 표현할 수 있는 식에서 계산 대상 값이 1인 경우,

덧셈과 뺄셈에 대해서는 다시 한번 축약할 수 있습니다.

\- 덧셈의 경우

```

x = x+1;

x +=1;

x++;

++x;

```

\-뺄셈의 경우

```

x = x -1;

x -=1;

x--;

--x;

```

\- x++ 와 ++x의 차이

\-> 증감 연산자는 그 자체가 다른 연산식의 피연산자로 사용될 수 있습니다.

이때, 증감연산자의 표시 위치에 따라 결과가 서로 다른게 적용된다.

\-x++

\-> 증감연산자가 뒤에 표시되는 경우, 현재 x의 값을 먼저 수식에 적용하고

나중에 x에 대한 1증가를 처리합니다.

```
int x= 100;

int a =1;

int y= a + x++;

System.out.println("y : " + y);
System.out.println("x : " + x);
```

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/6.png)

**y에 x가 대입될 때에는 x가 증가를 하기 전 100의 값으로 +a 가 되었다. 따라서 y에는 101이 대입되고 대입된 이후 x값은 증가하여 101이 된다.**

\-++x

\-> 증감연산자가 앞에 표시되는 경우, 먼저 현재 x의 값을 1증가 시킨 후, 그 결과를 수식에 적용한다.

```
int x= 100;

int a =1;

int y= a + ++x;

System.out.println("y : " + y);
System.out.println("x : " + x);
```

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/7.png)

**y에 값이 대입연산을 하기 전에 x값은 이미 증가를 했다. 따라서 y는 증가된 x값과 a값이 더해져서 102가 나온다.**

이번시간에는 여기까지 하겠습니다 ^^

많이 피곤하지만 배웠던것을 다지는 시간이라고 생각하니까 재밌네요~ 다음 시간에 봅시다.