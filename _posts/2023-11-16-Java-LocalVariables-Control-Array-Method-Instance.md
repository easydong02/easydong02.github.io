---
title: "[Java] 지역변수 제어문 배열 메서드 객체"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

**1\. 변수의 범위(=변수의 스코프)**

**1-1. 자신보다 하위 블록으로 침투할 수 있습니다.**

\-유효한 범위의 예

```

int num= 100;

if(num ==100){

// num이 현재 블록의 바깥에서 선언되었으므로 유효.

System.out.println(num);

}

```

\---------------------------------------------------------------------

```

int num= 100;

for( int i=0; i<10 ; i++){

System.out.println( num +i);

}

```

**1-2. 자신이 선언된 블록 밖으로는 빠져나갈 수 없습니다.**

\-허용되지 않는 범위의 예

```

int num =100;

if(num==100){

int result = num + 100;

}

//변수 result가 if블록 안에서 생성되었으므로 사용불가

System.out.println(result);

```

\--------------------------------------------------------------

```

for(int i =0; i<10; i++){

...

}

//i 가 for문을 위한 괄호 안에서 사용되었으므로 사용 불가.

System.out.println( i);

```

**1-3. 블록 안에서 선언된 변수는 블록 밖에서는 존재하는 동일한 이름의 변수와는 이름만 동일할 뿐, 다른 값으로 인식됩니다.**

```

int target = 100;


if(target == 100){

int num = target+100;

}else{

int num = target - 100;

}

```

// num은 서로 다른 블록에 존재하기 때문에 중복 선언 되더라도 유효합니다.

**2\. 여러가지 문법의 중첩 사용**

**2-1. 반복문의 흐름제어 기법**

\-break : 반복문 안에서 break 키워드를 만나면 반복을 강제로 종료합니다.

\-continue : 실행흐름이 증감식으로 강제 이동됩니다.

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/1.png)

```
// 1~100 홀수들의 합
		int sum=0;
		int i =0;
		
		while(true) {
			i++;
			if(i%2 ==0) {
				continue;
			}
			if(i==101) {
				break;
			}
			sum +=i;
		}
```

**3\. 배열**

**3-1. 배열이란?**

\- 변수를 그룹으로 묶는 형태의 한 종류로서, 사물함 같은 형태를 갖고 있습니다.

\- 하나의 배열 안에는 같은 종류(데이터 형)의 값들만 저장될 수 있습니다.

**3-2. 배열을 만드는 방법**

\- 배열의 선언

데이터형\[\] 배열이름;

\- 배열의 생성 - 변수를 저장할 수 있는 사물함을 생성합니다.

배열이름 = new 데이터형\[크기\];

\- 배열 생성의 예 -> 3개의 int형 변수를 저장할 수 있는 배열 생성

```

int[] grade; // 여러 개의 int형 변수를 저장할 수 있는 배열의 선언

grade = new int[3]; // 배열의 칸을 3칸으로 할당.
```

\- 배열의 선언과 크기 지정에 대한 일괄 처리

데이터형\[\] 배열이름 = new 데이터형\[크기\];

\- 배열 생성의 예 -> 3개의 int형 변수를 저장할 수 있는 배열 생성

```
int[] grade = new int[3];
```

**3-3. 배열에 값을 저장하기**

\- 배열은 값을 저장할 수 있는 공간일 뿐, 그 자체가 값은 아닙니다.

\- 값이 대입되지 않은 경우, 숫자형은 0, boolean형은 false가 자동으로 대입됩니다.

\- 배열 안에 값을 저장하기 위해서는 인덱스 번호를 사용하여 각각의 칸에 직접 값을

대입해야 합니다.

배열이름\[인덱스\] = 값;

\- 둘리의 점수를 배열로 표현한 예

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/2.png)

```

int[] grade = new int[3];

grade[0] = 75;

grade[1] = 82;

grade[2] = 91;
```

**3-4. 배열의 크기 설정과 값 할당에 대한 일괄처리**

\- 배열의 크기를 지정하면서 괄호 "{...}" 안에 배열에 포함될 각 항목들을

콤마(,)로 나열하면, 배열의 생성과 값의 할당을 일괄처리할 수 있습니다.

이 때는 배열의 크기를 별도로 지정하지 않으며, "new 데이터형\[\]" 부분은 생략 가능합니다.

데이터형\[\] 배열이름 = new 데이터형\[\]{값1, 값2, ..., 값n};

OR

데이터형\[\] 배열이름 = {값1, 값2, ..., 값n};

**3-5. 배열값 사용하기**

\- 배열 안에 저장되어 있는 값들을 사용하여 연산이나 출력 등의 처리를 위해서는

배열에 부여된 인덱스 값을 통해서 데이터에 접근해야 합니다.

```

System.out.println( grade[0]);

System.out.println( grade[1]);

System.out.println( grade[2]);
```

**3-6. 배열과 반복문**

\- 배열의 특성

\-> 0~ (배열크기 -1) 만큼의 인덱스 값을 순차적으로 갖습니다.

\- 특성을 활용한 배열 데이터의 처리

\-> 일정 범위를 갖고 순차적으로 증가하는 인덱스 값의 특성을 활용하면

반복문 안에서 배열의 값을 할당하거나, 할당된 값을 읽어들이는 처리가 가능합니다.

// 배열의 인덱스는 0부터 전체 길이 3보다 1작은 2까지입니다.

```

int[] grade = new int[]{ 100, 100, 90 };


for( int i =0; i<3 ; i++){

System.out.println( grade[i]);

}
```

**3-7. 배열의 크기(길이)**

\- 배열의 길이를 얻기 위해서는 "배열이름.length" 형식으로 접근합니다.

\- grade라는 배열을 생성한 경우 배열의 길이

```
int size = grade.length;
```

\- 배열의 길이값은 주로 반복문의 조건식에서 반복의 범위를 지정하기 위하여 사용됩니다.

```

int[] grade = new int[]{ 100, 100, 90 };


for( int i =0; i<grade.length ; i++){

System.out.println( grade[i]);

}
```

**3-8. 배열의 종류**

\-1차배열

\->앞에서 살펴본 배열처럼 한 줄만 존재하는 사물함 같이 구성된 배열

\-> 행에 대한 개념이 없고, 열에 대한 개념만 존재하기 때문에 배열이름.length는

몇칸인지 알아보는 기능이 된다.

\- 2차 배열

\-> 1차 배열의 각 칸에 새로운 배열을 넣는 형태

\-> 1차 배열의 각 칸은 행이 되고, 각각의 칸에 추가된 개별적인 배열이 "열"의

개념이 되어 "행렬"을 구성하게 됩니다.

**3-9. 2차원 배열의 선언**

\- 2차원 배열의 선언

\-> 데이터 타입의 이름 뒤에 대괄호 "\[\]"를 행과 열에 대하여 각각 지정합니다.

데이터형\[\]\[\] 배열이름;

\-2차원 배열의 크기 할당

\-> 행과 열에 대한 크기를 명시합니다.

배열이름 = new 데이터형\[행\]\[열\];

\- 2차원 배열의 선언과 할당의 일괄처리

데이터형\[\]\[\] 배열이름 = new 데이터형\[행\]\[열\];

\- 2차원 배열의 선언, 크기 할당, 값의 대입에 대한 일괄처리

\-> 2차원 배열의 경우 블록 괄호 '{}'를 2중으로 겹쳐서 2차원 배열을 표현합니다.

\-> 행과 열의 구분에는 콤마(,)가 사용됩니다.

\-> 컴파일러가 블록괄호 '{}'의 요소를 파악하면 행, 열의 크기가 산출될 수 있으므로

배열의 크기 설정을 위한 \[\]\[\]에는 배열의 크기를 명시하지 않습니다.

데이터형\[\]\[\] 배열이름 = new 데이터형 \[\]\[\]{

{0행},

{1행},

{2행},

{n행}

}

**3-10. 2차원 배열에 대한 값의 대입 방법**

\- 행, 열에 대한 인덱스를 통하여 값을 대입합니다.

행렬이름\[행\]\[열\] = 값;

\- 일괄지정하는 경우

```

int[][] grade = new int[][]{

{75, 82, 91},

{88, 64, 50},

{ 100, 100 , 90}

};
```

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/3.png)

**3-11. 2차원 배열의 길이**

\- 2차원 배열의 길이는 행에 대한 측면과 열에 대한 측면을 나누어서 생각해야 합니다.

\- 행의 길이

\-> 1차 배열의 길이는 2차 배열에서는 행의 크기로 조회됩니다.

```
int rows = grade.length;
```

\- 열의 길이

\-> 열의 길이는 각 행에 대하여 개별적으로 조회해야 합니다.

```
int cols = grade[행].length;
```

```
//구구단

for (int i = 2; i < 10; i++) {
			for (int j = 1; j < 10; j++) {
				System.out.println(i+"*" +j +"= " + i*j);
			}
			System.out.println("\n");
		}
```

```
//응용문제

/*
		 * ★★★★★★★★
		 *★★★★★★★★ 
		 * ★★★★★★★★
		 * ★★★★★★★★
		 * ★★★★★★★★
		 * ★★★★★★★★
		 * ★★★★★★★★
		 * ★★★★★★★★
		 */
		for (int i = 0; i < 8; i++) {
			for (int j = 0; j < 8; j++) {
				System.out.print("★");
			}
			System.out.print("\n");
		}
		
		/*
		 * ★★★★★★★★
		 *★★★★★★★
		 * ★★★★★★
		 * ★★★★★
		 * ★★★★
		 * ★★★
		 * ★★
		 * ★
		 */
		System.out.println("-----------------------");
		for (int i = 8; i > 0; i--) {
			for (int j = i; j >0; j--) {
				System.out.print("★");
			}
			System.out.println("");
		}
		/*
		 * ★
		 *★★
		 * ★★★
		 * ★★★★
		 * ★★★★★
		 * ★★★★★★
		 * ★★★★★★★
		 * ★★★★★★★★
		 */
		System.out.println("-----------------------");
		for (int i = 0; i <8; i++) {
			for (int j = 0; j <=i; j++) {
				System.out.print("★");
			}
			System.out.println("");
		}
```

**4\. 메서드**

**4-1. main()**

메인메서드는 저번에 말씀드린 것처럼 프로그램의 시작점입니다.

public static void main(String\[\] args){

}

**4-2. 메서드 만들기**

\- f(x) = x+1 이라는 함수는 다음의 정보를 구성하고 있습니다.

함수 이름: f

매개변수 : x

식 : x +1

f(1)= 2

f(3)= 4

\- 프로그램의 함수 = 메서드

\-> 특정 기능(=연산)을 그룹화해서 재사용하기 위한 단위

public static void 메서드 이름(){

.. 수행할 연산식...

}

```
public static void f() {
		int x =100;
		int y= x+1;
		System.out.println(y);
	}
```

\- 메서드의 호출

\-> 정의된 메서드는 다른 메서드를 구성하는 {...}안에서 다음의 형식으로 사용될

수 있으며, 이를 메서드를 호출한다고 합니다.

메서드이름();

```
public static void main(String[] args) {
		//재사용이 가능하다
		f();
}
```

**4-3. 수학의 매개변수**

\- 함수 f(x)는 주어지는 x값에 따라서 각각 다른 결과를 만들어냅니다.

\- 수학에서는 함수 f가 연산을 수행하기 위해서 주어지는 조건값을 매개변수라고 합니다.

f(x) =x+1;

\- 메서드의 파라미터

\-> java프로그램의 메서드(=함수)는 자신이 실행되는데 필요한 조건값을

메서드 이름 뒤의 괄호안에서 변수 형태로 선언한다. 이를 메서드 파라미터라고 합니다.

public static void 메서드이름( 변수형 변수이름){

}

\- 여러 개의 매개변수

\-> 특정 함수가 연산을 수행하기 위해서 두 개 이상의 조건값이 필요하다면

다음과 같이 콤마(,)로 구분하여 명시할 수 있습니다.

f(x,y) = x+y+1

\- 다중파라미터

\-> 메서드가 연산을 수행하는데 두 개 이상의 파라미터가 필요하다면 콤마(,)로

구분하여 선언할 수 있습니다.

public static void 메서드이름(변수형 변수이름, 변수형 변수이름){

...

}

```
	public static void f2(int x,int y) {
		int z= x *x + y +1;
		System.out.println(z);
	}
```

\- 파라미터를 갖는 메서드의 호출

\-> 메서드를 정의하면서 파라미터가 명시되어 있다면, 해당 메서드를 호출하면서

파라미터를 전달해 주어야 합니다.

public static void 메서드이름(변수형 변수이름, 변수형 변수이름){

...

}

```
f2(10,20);
```

\-> 메서드 호출하기

메서드이름(값1, 값2);

**4-4. 값을 반환하는 메서드**

\- 함수는 자신이 포함하고 있는 수식에 대한 결과를 반환합니다.

f(x) = x+1;

y = f(3); //4라는 결과값이 대입됩니다.

\- 메서드의 리턴값

\-> 메서드가 연산 결과를 자신이 호출된 위치에 반환하는 것을 "리턴"이라고 합니다.

\-> 메서드 안에서 값을 리턴하기 위해서는 "return"이라는 키워드를 사용해야 합니다.

\-> 값을 리턴하는 메서드는 선언시에 "void" 키워드 대신, 리턴하는 값에 대한

변수형이 명시됩니다. void는 리턴값이 없다는 의미입니다.

public static 리턴형 메서드이름( 변수형 파라미터1, .., 변수형 파라미터n){

return 리턴값;

}

```
//변수를 만들고 리턴값에 대입	
public static int f1( int x) {
		int y = x +1;
		return y;
	}
	
//리턴값에 바로 식 대입
	public static int f2(int x) {
		return x*x+1;
	}
```

**4-5. 메서드간의 상호 호출**

\- JAVA의 메서드 역시 서로 호출하는 것이 가능합니다. 호출된 메서드가 값을 리턴하는 경우,

리턴받은 값은 다른 연산에 사용할 수 있습니다.

public static 리턴형 메서드1이름(변수형 파라미터1){

return 리턴값;

}

public static 리턴형 메서드2이름(변수형 파라미터1){

int k = 메서드1이름(파라미터1);

return k;

}

```
	public static int f1(int x) {
		return x+1;
	}
	
	public static int f2(int x) {
		return f1(x);
	}
```

**5\. 객체지향 프로그래밍**

**5-1. 객체(=Object)**

\- 프로그래밍에서의 객체

\-> 프로그램에서 표현하고자 하는 기능을 묶기 위한 단위

\-객체지향 프로그래밍

\-> 객체가 중심이 되는 프로그래밍 기법

**5-2. 클래스와 객체의 관계**

\- 객체를 생성하기 위해서는 객체의 설계도가 필요하다 -> 클래스(Class)

\- 클래스(=Class) : 객체의 설계도 역할을 하는 프로그램 소스

\- 공장에서 하나의 설계도를 사용하여 여러 개의 제품을 생산할 수 있는 것처럼

하나의 클래스를 통해 동일한 구조를 갖는 객체를 여러개 생성할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/4.png)

**하나의 설계도(Class) 여러가지 제품을 만들 수 있다.**

**5-3. 객체를 구성하는 단위**

\- 객체를 이루는 것은 데이터와 기능입니다.

\-> 데이터는 변수로 표현합니다.

(객체 안에 포함된 변수를 '멤버변수' 혹은 '프로퍼티'라 합니다.

\-> 기능은 메서드(=함수)로 표현됩니다.

\- 자동차 클래스의 예

\-> 자동차의 엔진, 문, 바퀴, 등과 같이 명사적인 특성은 멤버변수로 존재할 수 있습니다.

\-> 전진, 후진 등과 같이 동사적인 특성은 메서드의 형태로 표현됩니다.

\-> 동일한 설계(Class)로 만들어진 자동차라 하더라도 각각의 자동차를 구성하는 부품들은 그 형태만 같은 뿐, 실제로는 각각 존재하게 됩니다.

\-> 클래스를 작성하면서 그 안에 생성되는 멤버변수들은 여러 개의 객체 간에 서로 동일한 이름으로 존재하지만, 실제로는 서로 다른 값이라는 의미.

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/5.png)

**같은 클래스라도 각 객체는 독립적인 멤버변수와 기능을 갖고 있습니다.**