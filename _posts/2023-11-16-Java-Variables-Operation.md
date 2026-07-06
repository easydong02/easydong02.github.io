---
title: "[Java] 변수(Variables) 연산자(Operation)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

3개월간의 선수과정을 마치고 국비 교육에 들어왔다. 하루 9시부터 6시까지, 선수과정보다 진도가 어마어마하다. 자바는 2달 정도 들었지만, **초심과 겸손**으로 처음 배우는 것처럼 다시 다져본다. 이번 글은 **프로그래밍 언어·JVM·주석·변수·자료형·연산자**를 폭넓게 정리하는 복습 편이다.

---

## 1. 프로그래밍 언어와 JVM

> **프로그래밍 언어**: 인간과 컴퓨터 사이의 의사소통을 가능케 하는 인공 언어. 명령어의 집합체인 **프로그램**을 작성한다.

| 종류 | 설명 |
|------|------|
| **기계어** | 컴퓨터가 이해하는 2진수의 집합 (사람은 이해 불가) |
| **고급언어** | 사람이 이해하는 수준. 기계어로 변환돼야 실행 (C, C++, Java) |

**Java의 특징:**

| 특징 | 설명 |
|------|------|
| **OS 독립적** | JVM만 설치되면 어디서든 실행 |
| **객체지향** | 상속·캡슐화·다형성 → 재사용·유지보수 용이 |
| **자동 메모리 관리** | Garbage Collector가 메모리 관리 |
| **네트워크·멀티스레드** | 손쉬운 API 제공 |

```
.java (소스)  ──컴파일러──►  .class (바이트코드)  ──JVM──►  각 OS
                            One Source, Multi Use
```

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/2.png)

> 💡 `.java`는 혼자 구동될 수 없다. 컴파일러가 `.class`로 변환하고, **JVM이 각 OS에 맞게 바이트코드를 전달**한다. 그래서 JVM만 있으면 하나의 프로그램이 환경에 상관없이 동일하게 실행된다.

---

## 2. 주석문

| 종류 | 문법 |
|------|------|
| 한 줄 주석 | `// 내용` |
| 여러 줄 주석 | `/* 내용 */` |

주석은 컴파일되지 않으며, 특정 명령을 실행되지 않게 차단하는 용도로도 쓴다.

```java
package HelloJava;

/*
 * 프로그램 소스의 최소 단위(=class)를 명시하는 블록.
 * 클래스 이름은 소스파일 이름과 동일, 첫 글자는 반드시 영어.
 */
public class HelloJava {
    // 프로그램의 시작점 → main 메서드
    public static void main(String[] args) {
        System.out.println("Hello Java");   // 콘솔 출력
    }
}
```

> 💡 무심코 지나쳤던 `main` 메소드가 사실 **프로그램의 시작점을 의미하는 블록**이었다. 제대로 알고 보니 반갑다.

---

## 3. 변수와 자료형

> **자료형(data type)**: 변수의 종류를 구별하는 키워드. 자바의 **기본 자료형(Primitive)은 8가지**다.

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/3.png)

> ⚠️ 모든 변수는 **RAM**에 생성된다. 4byte 변수를 만들면 RAM에서 그만큼 사용한다. RAM 크기를 넘으면 **OutOfMemory** 에러가 난다. (혼자 작업할 땐 볼 일 없지만, 실무의 공유 프로그램에서는 메모리 초과가 발생하기도 한다.)

### 변수의 선언과 할당

```java
int num1;        // 선언 (데이터형 + 이름)
num1 = 100;      // 할당 (= 로 값 대입, 우변 → 좌변)

int num2 = 1000; // 선언 + 할당 통합
```

**문자열 데이터:**

```java
String msg = "안녕하세요. 자바 ";   // 공백 포함
String blank = "";                  // 빈 문자열
String age = "22";                  // 쌍따옴표 안이면 숫자도 문자열

String language = "JA" + "VA";      // "JAVA" (문자열 + 문자열)
String result = "자바학생" + 20;    // "자바학생20" (문자열 + 기본형)
```

### 변수 사용의 제약

| 규칙 | 예 |
|------|------|
| 값 재할당은 가능, **선언 중복은 불가** | `int num1=200; ... int num1=300; // 에러` |
| **선언 안 된 변수** 사용 불가 | `num2 = 700; // 에러` |
| **할당 안 된 변수** 대입·출력 불가 | `int b = num1; // num1 미할당이면 에러` |

**변수명 규칙:** 영문·숫자·`_`·`$`만 가능, 숫자로 시작 불가, 대소문자 구별(`name ≠ Name`), 예약어 사용 불가.

---

## 4. 값의 할당과 상수

```java
boolean isKorean = true;          // true / false
char first = '곽';                // 홑따옴표 한 글자
long money = 5000000L;            // 접미사 L
float pi = 3.14F;                 // 접미사 F
double lat = 128.32452D;          // 접미사 D
```

> **상수(constant)**: `final` 키워드로 선언하며 **값을 변경할 수 없는(읽기 전용)** 데이터. 관례상 이름을 대문자로 쓴다.

```java
final int AGE = 22;
AGE = 23;   // ❌ 상수 값 변경 → 에러
```

---

## 5. 연산자

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/5.png)

### 사칙 연산자

```java
int num1 = 12, num2 = 8;
num1 + num2;   // 20
num1 - num2;   // 4
num1 * num2;   // 96
num1 / num2;   // 1  (몫)
num1 % num2;   // 4  (나머지)
```

> ⚠️ **나눗셈 주의**: `10 / 3 = 3`(몫), `10 % 3 = 1`(나머지). 정수+실수 연산은 실수로 자동 변환된다. **모든 수는 0으로 나눌 수 없다.** 우선순위는 `* / %` > `+ -`, 괄호가 최우선.

### 단항(복합 대입) 연산자

```java
x = x + 5;   →   x += 5;   // +=, -=, *=, /=, %= 모두 가능
```

### 증감 연산자 — x++ vs ++x

```java
x = x + 1;  →  x += 1;  →  x++;  또는  ++x;
```

증감 연산자가 **다른 수식의 피연산자**로 쓰일 때 위치에 따라 결과가 다르다.

```java
int x = 100, a = 1;
int y = a + x++;   // 후위: 현재 x(100)를 먼저 쓰고 나중에 증가
// y = 101, x = 101
```

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/6.png)

```java
int x = 100, a = 1;
int y = a + ++x;   // 전위: 먼저 x를 증가시킨 뒤 사용
// y = 102, x = 101
```

![Desktop View](/assets/img/Programming-Language/Java/Variable-Operation/7.png)

| 표기 | 동작 | y | x |
|:---:|------|:---:|:---:|
| `a + x++` | x를 쓴 뒤 증가 | 101 | 101 |
| `a + ++x` | x를 증가 후 사용 | 102 | 101 |

---

## 📝 정리

```
변수 & 연산자 (복습)
├─ JVM      .java → .class → JVM → 각 OS (One Source Multi Use)
├─ 주석      // 한 줄, /* 여러 줄 */
├─ 변수      선언 + 할당, RAM에 생성 (OutOfMemory 주의)
├─ 자료형    기본형 8가지, 접미사 L/F/D
├─ 상수      final, 읽기 전용
└─ 연산자    사칙(/, %) · 복합대입(+=) · 증감(x++ vs ++x)
```

| 개념 | 한 줄 정의 |
|------|------|
| **JVM** | 바이트코드를 각 OS에서 실행 |
| **자료형** | 변수의 종류·크기를 정하는 키워드 |
| **final** | 값을 못 바꾸는 상수 |
| **x++ vs ++x** | 사용 후 증가 vs 증가 후 사용 |

기본기를 다시 훑으니, 무심코 넘어갔던 `main`의 의미나 `x++`/`++x`의 차이가 새롭게 다가온다. 피곤하지만 기초를 다지는 시간이라 오히려 재미있다.
