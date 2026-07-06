---
title: "[Java] 배열"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

변수를 여러 개 선언하면 관리가 어렵다. 이 여러 저장 공간을 **하나의 이름**으로 묶는 것이 **배열(array)**이다. C에서도 봤지만, 자바의 배열은 **`new`로 Heap 메모리에 할당**된다는 특징이 있다.

> **배열 = 저장 공간의 나열**
> - 여러 변수를 하나의 이름으로 묶어 **관리가 편하다.**
> - 규칙성 없는 값에 **규칙성(인덱스)을 부여**한다.

---

## 1. 배열의 선언

```java
자료형[] 배열명 = {값1, 값2, 값3, ...};   // 값으로 초기화
자료형[] 배열명 = new 자료형[칸수];        // 크기로 선언
자료형[] 배열명;
배열명 = new 자료형[칸수];                 // 선언과 할당 분리
```

C와 다른 두 가지 포인트가 있다.

| 항목 | C | Java |
|------|------|------|
| 대괄호 위치 | `자료형 배열명[]` | **`자료형[] 배열명`** |
| 메모리 | 스택 등 | **`new`로 Heap에 할당** |

> 💡 `new`는 **Heap 메모리에 저장**하는 연산자다. 그렇다면 `{ }`만 쓰면 Heap에 안 들어갈까? 아니다. **중괄호는 `new`를 생략한 것뿐**이다. 따라서 자바의 배열은 모두 Heap에 들어가는 **동적 배열**이다.

---

## 2. 배열의 길이(length)와 출력

배열 선언 시 길이가 **`length` 상수**에 담긴다. `배열명.length`로 사용한다.

```java
int[] arData = {3, 4, 6, 7};

System.out.println(arData);                     // 주소값 (arData[0]의 주소)
System.out.println(arData[0]);                  // 3   (첫 번째 값)
System.out.println(arData[arData.length - 1]);  // 7   (마지막 칸)
System.out.println(arData.length);              // 4   (길이)

for (int i = 0; i < arData.length; i++) {
    System.out.println(arData[i]);
}
```

![Desktop View](/assets/img/Programming-Language/Java/Array/1.png)

| 코드 | 결과 | 의미 |
|------|------|------|
| `arData` | 이상한 값 | `arData[0]`의 **주소값** |
| `arData[0]` | 3 | 첫 번째 값 |
| `arData.length - 1` | 3(인덱스) | **마지막 칸**을 가리킴 |
| `arData.length` | 4 | 배열 길이 |

### index가 0부터 시작하는 이유

> 배열은 **시작 주소**를 갖고, 다음 칸으로 갈 때마다 시작 주소에 `+1`씩 더한다. 첫 번째 방은 `시작주소 + 0`이고, 이 `0`이 그대로 인덱스가 된다. 그래서 **첫 칸의 인덱스는 0**이다. (`for`문의 `i=0`이 여기서 유래한다.)

---

## 3. 배열에 값 입력하기

```java
int[] arData = new int[5];

for (int i = 0; i < arData.length; i++) {
    arData[i] = 5 - i;      // 5, 4, 3, 2, 1
}
for (int i = 0; i < arData.length; i++) {
    System.out.println(arData[i]);
}
```

![Desktop View](/assets/img/Programming-Language/Java/Array/2.png)

---

## 4. 실습

### ① 100~1 누적합

```java
int[] arData = new int[100];
int sum = 0;
for (int i = 0; i < arData.length; i++) {
    arData[i] = 100 - i;    // 100, 99, ..., 1
    sum += arData[i];       // 누적
}
System.out.println(sum);
```

### ② 최대값·최소값

```java
int[] arData = new int[5];
Scanner sc = new Scanner(System.in);
int max = 0, min = 0;

for (int i = 0; i < arData.length; i++) {
    System.out.print((i + 1) + "번째 정수 : ");
    arData[i] = sc.nextInt();     // 5개 입력받기
}

max = arData[0];                  // 첫 값을 후보로
min = arData[0];
for (int i = 1; i < arData.length; i++) {   // 인덱스 1부터 비교
    if (max < arData[i]) max = arData[i];
    if (min > arData[i]) min = arData[i];
}
System.out.println("최대값 : " + max);
System.out.println("최소값 : " + min);
```

> 💡 `for`가 **1부터** 시작하는 이유: 첫 번째 값(`arData[0]`)은 이미 `max`/`min`에 넣었으므로, 인덱스 1부터 비교하면 된다.

### ③ 정수를 한글로 변환 (예: 1024 → 일공이사)

```java
String hangle = "공일이삼사오육칠팔구";
Scanner sc = new Scanner(System.in);
int num = 0;
String result = "";

System.out.print("정수 입력 : ");
num = sc.nextInt();

while (num != 0) {
    result = hangle.charAt(num % 10) + result;   // 뒤에서부터 앞으로 붙임
    num /= 10;
}
System.out.println(result);
```

![Desktop View](/assets/img/Programming-Language/Java/Array/3.png)

동작 원리를 뜯어보면 이렇다.

| 요소 | 설명 |
|------|------|
| `charAt(i)` | 문자열은 클래스라 `[i]` 대신 `charAt(i)`로 문자 접근 |
| `num % 10` | 일의 자리 추출 (1024 → 4 → `charAt(4)` = "사") |
| `+ result` | **앞에 붙여** 순서를 맞춤 (안 그러면 "사이공일") |
| `num /= 10` | 자리 하나 제거 (1024 → 102), num이 0이면 종료 |

---

## 📝 정리

```
Java 배열
├─ 선언    자료형[] 이름 = {...} 또는 new 자료형[n]
├─ 메모리  new로 Heap에 할당 (동적 배열)
├─ length  배열명.length (마지막 칸 = length-1)
├─ 인덱스  0부터 시작 (시작주소 + 0)
└─ 활용    누적합, 최대/최소, charAt로 문자 접근
```

| 개념 | 한 줄 정의 |
|------|------|
| **length** | 배열 길이를 담는 상수 |
| **new** | Heap 메모리에 배열을 할당하는 연산자 |
| **charAt(i)** | 문자열의 i번째 문자에 접근 |

배열은 여러 저장 공간을 하나로 묶는 기본 자료구조다. 자바에서는 `new`로 Heap에 할당된다는 점, 그리고 인덱스가 0부터 시작하는 이유(시작 주소 + 0)를 이해하면 응용 문제도 차근차근 풀 수 있다.
