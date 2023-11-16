---
title: 연산자 형변환 조건문 반복문
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

**1\. 연산자**

**1-1. 비교 연산자**

\- 같다, 다르다 크다(초과), 크거나 같다(이상), 작다(미만), 작거나 같다(이하)에 대한 비교를

수행하는 연산자입니다.

같다 ==

다르다 !=

초과 >

이상 >=

미만 <

이하 <=

\-주어진 식이 참인지 거짓인지만 판별 가능하므로, 연산 결과는 boolean 값으로 생성됩니다.

```

int x= 100;

int y= 1;

boolean r = x (비교연산자) y;
```

**1-2. 논리 연산자(&&)**

\- 두 개 이상의 비교 연산자의 결과나 boolean 값에 대해 추가로 "AND"나 "OR"연산을

수행하여 결과를 얻습니다.

\- AND의 의미를 갖는 &&

\- "&&"연산자는 두 개의 boolean 값을 비교하여 모두 true인 경우만 결과값이 true가 나옵니다.

![Desktop View](/assets/img/Programming-Language/Java/Operation-Cast-Condition-Loop/1.png)

**1-3. 논리 연산(||)**

\- "||" 연산자는 두 개의 boolean 값을 비교하여 둘 중 하나라도 true인 경우

결과가 true가 나옵니다.

![Desktop View](/assets/img/Programming-Language/Java/Operation-Cast-Condition-Loop/2.png)

```
		int a= 100;
		int b = 200;
		int x= 5;
		int y= 3;

        boolean r1 =a >=b;	//false
		boolean r2 = x >y;	//true
		boolean result6 = r1 &&r2;	//false
		boolean result7 = r1 ||r2;	//true
```

**2\. 형변환(casting)**

**2-1. 형변환**

\- 변수의 자료형이 변환되는 것을 의미합니다.

\- 특정 자료형의 값을 다른 자료형의 변수에 대입할 수 있습니다.

\- 암묵적 형변환과 명시적 형변환 두 종류가 있습니다.

**2-2. 암묵적 형변환**

\- 서로 다른 자료형을 연산 혹은 대입하는 경우, Java 컴파일러가 자료형을 통일합니다.

이 과정에서 발생하는 형변환을 암묵적 형변환이라고 합니다.

![Desktop View](/assets/img/Programming-Language/Java/Operation-Cast-Condition-Loop/3.png)

```

long a = 100; //정수형

float b = a; //정수형을 실수형에 대입

```

\-> 암묵적 형변환은 어떤 변수가 더 큰 범위의 변수로 대입 가능함을 의미합니다.

\- 암묵적 형변환은 데이터의 손실이 발생하지 않는 범위 내에서만 이루어집니다.

\- double 형 데이터 20.5를 int에 대입하는 경우에는 0.5에 대한 데이터 손실이 불가피하므로,

에러가 발생합니다.

**2-3. 명시적 형변환**

\- 데이터의 손실은 감수하더라도, 강제로 형변환 시키는 형태

자료형 b = (반환할 자료형) a;

\-실수형을 정수형으로 변환하는 경우, 소수점 이하 자리는 버려진다.

```

double a = 3.14d;

int b =a;

```

```
double a = 10.5;
		float b= 20.5f;
		
		/*
		 * 큰 범위의 변수와 작은 범위의 변수가 연산을 수행하면,
		 * 작은 범위의 변수가 큰 범위의 데이터형으로 암묵적 형변환을 수행한다.
		 * 그러므로 아래의 a+b는 double형의 변수가 되므로,
		 * float형의 값에 대입하는 것은 에러다.
		 */
		
		float f = a +b;  //에러
        double f = a+b;
```

**3.조건문**

**3-1. 조건문이란?**

\- 무조건 실행되는 것이 아니라, 특정 조건을 충족할 경우에만 실행되는 구문입니다.

**3-2. 조건문의 분류**

종류 설명

if문 주어진 '조건'이 참(true)일 경우에만 실행됩니다.

if~else문 주어진 조건이 참일 경우 if문이 실행되고, 그렇지 않을 경우 else문이 실행됩니다.

if~else if~else문 조건을 여러 개로 세분화하여 사용하는 if문입니다.

switch문 하나의 '값'에 대하여 여러가지 경우의 수로 나누어 분기합니다.

**3-3. if문**

\- if문은 주어진 조건이 참일 경우에 지정된 구문이 실행된다.

```
if(조건){

.. 실행할 구문 ..

}
```

\-실행할 구문이 한 줄만 있을 경우 괄호{}는 생략 가능합니다.

```
if(조건)

...실행할 구문...
```

\-if문의 조건식

\->비교식(비교 연산자 사용)

\->논리식(논리 연산자 사용)

\->boolean값

```
if(myAge>19)
if(point>70 && point<=80)
boolean is_korean = true;
if(is_korean)
```

**3-4. if ~ else문**

\-그렇지 않으면?

\- if문이 조건이 참일 경우에 실행되는 구문이라면, if문의 조건과 반대되는 경우에 실행되는 구문이 else문입니다.

\- else문은 독립적으로 실행될 수 없고, 반드시 if문의 뒤에 위치해야합니다.

```

if(조건){

..실행할 구문..

} else{

..반대의 경우에 실행할 구문..

}
```

```

boolean is_korean = true;

if( is_korea ==true){

System.out.println("한국사람입니다.");

} else{

System.out.println("한국사람이 아닙니다.");

}
```

**3-5. if ~ else if ~ else문**

\- if문과 else문 사이에 else if 문으로 두 번째 조건, 세 번째 조건을 나열할 수 있습니다.

\- else if 문은 필요한 만큼 나열할 수 있으며, 필요치 않을 경우 else 문은 생략 가능합니다.

```

if(1차조건){

..실행할 구문..

} else if(2차 조건){

..실행할 구문..

} else if(n차 조건){

..실행할 구문..

} else{

..실행할 구문..

}

```

**3-6. switch문**

\- switch문은 하나의 변수(기준값)에 대한 여러가지 case를 정의하는 구문입니다.

\- if 문은 조건에 식(비교식,부등식)이 사용될 수 있지만 switch문은 분기 조건이 반드시 일치하는 "값"에 대해서만 처리 가능합니다.

```

switch(기준값){

case 값1 :

..실행할 구문..

break;



case 값2 :

..실행할 구문..

break;


case 값3 :

..실행할 구문..

break;


default :

.. 모든 경우에 충족되지 않을 경우 실행될 기본 구문..

break;

}

```

**4\. 반복문**

**4-1. for문**

\-for문은 사람이 직접 처리하기에 부담스러운 반복적인 작업을 처리하기에 매우 용이합니다.

```
//7의 배수
		int j =0;
		for (int i = 1; i < 10; i++) {
			j= 7*i;
			System.out.println("j : "+ j);
		}
```

**4-2. while문**

\-while문 역시 for문과 마찬가지로 반복적인 처리를 수행하는 문법입니다.

\-for문은 초기식, 조건식, 증감식을 모두 내장하는 반면, while문은 조건식만을 내장하기 때문에 초기식과 증감식을 외부에 따로 정의해 주어야 합니다.

```
        int j=0;
		int i =1;
		while(i<10) {
		j=i*7;
		i++;
		System.out.println(j);
		}
```

**4-3. do ~ while문**

\- 조건의 판별을 나중에 수행하는 반복문 형태

\- 초기식을 설정한 후 do{..}안의 문장을 우선적으로 1회 실행하고 조건을 판별하므로, 조건이 참이 아니더라도 최소 1회는 실행됩니다.

```
		int sum =0;
		int i =1;
		
		do {
			sum +=i;
			i++;
		}while(i<=100);
		
		System.out.println(sum);
```

**4-4. 무한 루프**

\- 증감식이 설정되지 않거나, 증감식이 수행되더라도 조건식이 거짓이 되지 않는 형태

\- 조건식이 항상 참이므로, 반복문이 종료되지 않습니다.

\- 프로그램이 PC의 자원을 매우 많이 사용하게 되므로, 시스템 다운을 발생시킬 수 있습니다.

\- for문의 무한 루프 예

```

for( int i =0; i<10;i--){


}

```

\- while문의 무한 루프

```

whlie(true){

System.out.println("hello");

}

```

**문제**

```
/*
		 * 1. 1~10까지의 합
		 */
		int sum =0;
		for (int i = 0; i <= 10; i++) {
			sum +=i;
		}
		System.out.println("1~10까지의 합"+sum);
		
		System.out.println("--------------------");
		/*
		 * 2. 1~10까지의 홀수값의 합
		 */
		sum=0;
		for (int i = 0; i <= 10; i++) {
			if(i%2 ==1) {sum +=i;}
		}
		System.out.println("1~10까지의 홀수값의 합"+sum);
		
		System.out.println("--------------------");
		/*
		 * 3. 1~10까지 짝수값의 합
		 */
		sum=0;
		for (int i = 0; i <= 10; i++) {
			if(i%2 ==0) {sum +=i;}
		}
		System.out.println("1~10까지의 짝수값의 합"+sum);
		System.out.println("--------------------");
		
		/*
		 * 4. 10번찍어 안 넘어 가는 나무 없다.
		 * 나무를 1번 찍었습니다.
		 * ...
		 * 
		 * 나무를 10번 찍었습니다.
		 * 나무가 넘어갑니다.
		 */
		for (int i = 1; i < 11; i++) {
			System.out.println("나무를 " + i + "번 찍었습니다.");
			if(i==10) {
				System.out.println("나무가 넘어갑니다.");
			}
		}
```