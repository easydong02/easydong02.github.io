---
title: "[Java] 지역변수 제어문 배열 메서드 객체"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글은 **변수 스코프**, **제어문 흐름 제어(break/continue)**, **배열(1차·2차)**, **메서드**, **객체지향**까지 폭넓게 정리하는 복습 편이다.

---

## 1. 변수의 범위(스코프)

변수 스코프의 세 가지 규칙을 정리하면 이렇다.

| 규칙 | 설명 |
|------|------|
| **하위 블록 침투 가능** | 바깥에서 선언된 변수는 안쪽 블록에서 사용 가능 |
| **상위 블록 이탈 불가** | 안쪽 블록에서 선언된 변수는 밖에서 사용 불가 |
| **다른 블록끼리는 독립** | 다른 블록의 같은 이름 변수는 별개 (중복 선언 가능) |

```java
// ✅ 하위 블록으로 침투 가능
int num = 100;
if (num == 100) { System.out.println(num); }   // OK

// ❌ 블록 밖으로 이탈 불가
if (num == 100) { int result = num + 100; }
System.out.println(result);   // 에러 (result는 if 안에서만)
for (int i = 0; i < 10; i++) {}
System.out.println(i);         // 에러 (i는 for 안에서만)

// ✅ 다른 블록끼리는 중복 선언 OK
if (target == 100) { int n = target + 100; }
else               { int n = target - 100; }   // 서로 다른 블록
```

---

## 2. 반복문 흐름 제어

| 키워드 | 동작 |
|------|------|
| **break** | 반복을 강제 종료 |
| **continue** | 증감식(다음 반복)으로 강제 이동 |

```java
// 1~100 홀수들의 합
int sum = 0, i = 0;
while (true) {
    i++;
    if (i % 2 == 0) continue;   // 짝수는 건너뜀
    if (i == 101)   break;      // 종료
    sum += i;
}
```

---

## 3. 배열

> **배열**: 같은 데이터형의 값들을 그룹으로 묶은 **사물함** 같은 구조.

```java
int[] grade;              // 선언
grade = new int[3];       // 생성 (3칸)
int[] grade2 = new int[3];   // 선언+생성 통합
```

> 💡 배열은 **값을 담는 공간**일 뿐 그 자체가 값은 아니다. 값을 안 넣으면 숫자형은 0, boolean은 false로 자동 초기화된다.

**값 저장·초기화:**

```java
grade[0] = 75; grade[1] = 82; grade[2] = 91;   // 인덱스로 대입

int[] g = new int[]{75, 82, 91};   // 생성+할당 일괄
int[] g2 = {75, 82, 91};           // new 생략 가능
```

**배열과 반복문** — 인덱스가 `0 ~ (길이-1)`로 순차 증가하는 특성을 활용한다.

```java
int[] grade = {100, 100, 90};
for (int i = 0; i < grade.length; i++) {   // length로 범위 지정
    System.out.println(grade[i]);
}
```

### 2차원 배열

> **2차원 배열**: 1차 배열의 각 칸에 또 배열을 넣은 형태. 1차 배열의 칸이 **행**, 그 안의 배열이 **열**이 되어 행렬을 구성한다.

```java
int[][] grade = new int[][]{
    {75, 82, 91},
    {88, 64, 50},
    {100, 100, 90}
};
```

| 길이 | 접근 방법 |
|------|------|
| 행의 개수 | `grade.length` |
| 특정 행의 열 개수 | `grade[행].length` |

**응용 — 별 찍기:**

```java
// 정사각형
for (int i = 0; i < 8; i++) {
    for (int j = 0; j < 8; j++) System.out.print("★");
    System.out.println();
}

// 역삼각형 (8→1)
for (int i = 8; i > 0; i--) {
    for (int j = i; j > 0; j--) System.out.print("★");
    System.out.println();
}

// 삼각형 (1→8)
for (int i = 0; i < 8; i++) {
    for (int j = 0; j <= i; j++) System.out.print("★");
    System.out.println();
}
```

---

## 4. 메서드

> **메서드 = 프로그램의 함수.** 특정 기능을 그룹화해 재사용하는 단위. 수학의 `f(x) = x+1`과 같은 구조다.

```java
// 파라미터 없음
public static void f() {
    int x = 100;
    System.out.println(x + 1);
}

// 파라미터 있음 (여러 개는 콤마로)
public static void f2(int x, int y) {
    System.out.println(x * x + y + 1);
}
f2(10, 20);   // 호출 시 값 전달
```

**값을 반환하는 메서드** — `void` 대신 리턴형을 명시하고 `return`을 쓴다.

```java
public static int f1(int x) {   // int 반환
    int y = x + 1;
    return y;
}
public static int f2(int x) {
    return x * x + 1;           // 식을 바로 리턴
}
```

**메서드 상호 호출** — 리턴받은 값을 다른 연산에 쓸 수 있다.

```java
public static int f1(int x) { return x + 1; }
public static int f2(int x) { return f1(x); }   // f1을 호출
```

---

## 5. 객체지향 프로그래밍

> **객체(Object)**: 표현하고자 하는 기능을 묶는 단위. **객체지향**은 객체가 중심이 되는 프로그래밍 기법.

**클래스와 객체의 관계** — 클래스는 **설계도**, 객체는 그 설계도로 찍어낸 **제품**이다.

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/4.png)

**객체를 구성하는 단위:**

| 구성 | 표현 | 자동차 예 |
|------|------|------|
| **데이터** | 멤버변수(프로퍼티) | 엔진, 문, 바퀴 (명사) |
| **기능** | 메서드 | 전진, 후진 (동사) |

![Desktop View](/assets/img/Programming-Language/Java/LocalVariable-Control-Array-Method-Instance/5.png)

> 💡 같은 클래스로 만든 객체라도 각 부품(멤버변수)은 형태만 같을 뿐 **독립적으로 존재**한다. 즉 여러 객체의 멤버변수는 이름은 같아도 값은 서로 다르다.

---

## 📝 정리

```
지역변수·제어·배열·메서드·객체
├─ 스코프    하위 침투 O, 상위 이탈 X, 다른 블록은 독립
├─ 제어      break(종료) / continue(다음 반복)
├─ 배열      1차(length) / 2차(행 length, 열 [행].length)
├─ 메서드    기능 그룹화·재사용, 파라미터·return
└─ 객체      클래스(설계도) → 객체(제품), 데이터+기능
```

| 개념 | 한 줄 정의 |
|------|------|
| **변수 스코프** | 변수가 유효한 블록 범위 |
| **2차원 배열** | 배열 안의 배열 (행렬) |
| **메서드** | 기능을 그룹화한 재사용 단위 |
| **클래스 vs 객체** | 설계도 vs 찍어낸 제품 |

변수 스코프부터 객체지향까지 한 번에 훑었다. 특히 **배열 인덱스는 0부터 length-1**, **클래스는 설계도·객체는 제품**이라는 두 축을 잡으면 이후 내용이 술술 연결된다.
