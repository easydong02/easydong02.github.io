---
title: "[Java] 연산자 형변환 조건문 반복문"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글은 **비교·논리 연산자**, **형변환(암묵적/명시적)**, **조건문(if/switch)**, **반복문(for/while/do-while)**을 폭넓게 정리하는 복습 편이다.

---

## 1. 비교·논리 연산자

**비교 연산자**는 참/거짓만 판별하므로 결과가 **boolean**이다.

| 연산자 | 의미 | | 연산자 | 의미 |
|:---:|------|---|:---:|------|
| `==` | 같다 | | `<` | 미만 |
| `!=` | 다르다 | | `>=` | 이상 |
| `>` | 초과 | | `<=` | 이하 |

**논리 연산자**는 두 개 이상의 boolean을 AND/OR로 결합한다.

| 연산자 | 참이 되는 조건 |
|:---:|------|
| `&&` (AND) | 둘 다 true |
| `\|\|` (OR) | 하나라도 true |

```java
int a = 100, b = 200, x = 5, y = 3;
boolean r1 = a >= b;      // false
boolean r2 = x > y;       // true
boolean result6 = r1 && r2;   // false (AND)
boolean result7 = r1 || r2;   // true  (OR)
```

---

## 2. 형변환 (Casting)

| 종류 | 설명 |
|------|------|
| **암묵적 형변환** | 컴파일러가 자동으로 자료형을 통일 (**손실 없는 범위**에서, 작은→큰) |
| **명시적 형변환** | 손실을 감수하고 `(자료형)`으로 강제 변환 |

```java
// 암묵적 (작은 범위 → 큰 범위, 손실 없음)
long a = 100;
float b = a;

// 명시적 (큰 범위 → 작은 범위, 소수점 버려짐)
double c = 3.14d;
int d = (int) c;   // 3
```

> ⚠️ `double` 20.5를 `int`에 그냥 대입하면 0.5 손실이 불가피해 **에러**가 난다. 강제 변환 `(int)`을 써야 한다. 또, 큰 범위와 작은 범위가 연산하면 결과는 **큰 범위**가 되므로 주의한다.

```java
double a = 10.5;
float b = 20.5f;
float f = a + b;   // ❌ 에러 (a+b는 double)
double f2 = a + b; // ✅
```

---

## 3. 조건문

| 종류 | 설명 |
|------|------|
| `if` | 조건이 참일 때만 실행 |
| `if ~ else` | 참이면 if, 아니면 else |
| `if ~ else if ~ else` | 조건을 여러 개로 세분화 |
| `switch` | 하나의 **값**에 대해 여러 case로 분기 |

```java
// if의 조건식: 비교식 / 논리식 / boolean값
if (myAge > 19) ...
if (point > 70 && point <= 80) ...
boolean is_korean = true;
if (is_korean) ...
```

> 💡 `if`는 식(비교·부등식)을 조건으로 쓸 수 있지만, **`switch`는 반드시 일치하는 '값'**에 대해서만 분기한다. 각 case는 `break`로 마무리하고, 어디에도 안 맞으면 `default`가 실행된다.

---

## 4. 반복문

| 반복문 | 특징 |
|------|------|
| `for` | 초기식·조건식·증감식을 **모두 내장** |
| `while` | **조건식만** 내장 (초기식·증감식은 외부에) |
| `do ~ while` | 조건을 나중에 판별 → **최소 1회 실행** |

```java
// for
for (int i = 1; i < 10; i++) {
    System.out.println("j : " + (7 * i));
}

// while (초기식·증감식 외부)
int i = 1;
while (i < 10) {
    System.out.println(i * 7);
    i++;
}

// do~while (최소 1회)
int sum = 0, k = 1;
do {
    sum += k;
    k++;
} while (k <= 100);
System.out.println(sum);
```

> ⚠️ **무한 루프** 주의: 증감식이 없거나 조건이 절대 거짓이 안 되면 종료되지 않아 시스템 자원을 과도하게 쓴다. (예: `for(int i=0; i<10; i--)`, `while(true){...}`)

---

## 5. 연습 문제

```java
// 1. 1~10까지의 합
int sum = 0;
for (int i = 0; i <= 10; i++) sum += i;
System.out.println("1~10까지의 합 " + sum);          // 55

// 2. 홀수 합 / 3. 짝수 합
sum = 0;
for (int i = 0; i <= 10; i++) if (i % 2 == 1) sum += i;  // 홀수
sum = 0;
for (int i = 0; i <= 10; i++) if (i % 2 == 0) sum += i;  // 짝수

// 4. 열 번 찍어 안 넘어가는 나무 없다
for (int i = 1; i < 11; i++) {
    System.out.println("나무를 " + i + "번 찍었습니다.");
    if (i == 10) System.out.println("나무가 넘어갑니다.");
}
```

---

## 📝 정리

```
연산자·형변환·조건·반복
├─ 비교/논리  결과는 boolean, && (둘 다) / || (하나라도)
├─ 형변환     암묵적(작은→큰) / 명시적((int) 손실 감수)
├─ 조건문     if / if-else / else if / switch(값 분기)
└─ 반복문     for / while / do-while(최소 1회) / 무한루프 주의
```

| 개념 | 한 줄 정의 |
|------|------|
| **암묵적 형변환** | 손실 없는 자동 변환 (작은→큰) |
| **명시적 형변환** | `(타입)`으로 강제 (손실 감수) |
| **switch** | 값 일치로 분기 (break 필수) |
| **do-while** | 조건 상관없이 최소 1회 실행 |

기본기를 한 번에 훑는 복습이었다. 특히 형변환에서 **연산 결과가 큰 범위 타입이 된다**는 규칙, 그리고 무한 루프 조심만 잊지 않으면 탄탄한 기초가 된다.
