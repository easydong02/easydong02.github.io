---
title: "[Java] 형변환 연산 입력"
date: 2023-11-12 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **형변환**을 마저 다지고, **문자열↔숫자 변환**, **입력(Scanner)**, 그리고 **비트·단항·쉬프트 연산자**까지 정리한다.

---

## 1. 문자 + 정수 vs 문자열 + 정수

핵심 규칙 두 가지를 먼저 잡자.

| 식 | 결과 | 동작 |
|------|------|------|
| **문자 + 정수** | 정수 | **연산** (아스키코드로 계산) |
| **문자열 + 값** | 문자열 | **연결**(concatenation) |

```java
int data = 'A';                       // 'A' → 아스키코드 65
System.out.println(data);             // 65
System.out.println((char)('A' + 5));  // 'A'+5=70 → char → 'F'
```

```java
String data1 = "1";
String data2 = "2";
System.out.println(data1 + data2);    // "12" (연산이 아니라 연결!)
```

---

## 2. 문자열 ↔ 숫자 변환

`(int)data1`처럼 강제 형변환은 **안 된다.** `String`은 클래스이기 때문이다. 대신 전용 메소드를 쓴다.

| 변환 | 메소드 |
|------|------|
| 문자열 → 정수 | `Integer.parseInt(문자열)` |
| 문자열 → 실수 | `Double.parseDouble(문자열)` |

```java
int num1 = Integer.parseInt(data1);
int num2 = Integer.parseInt(data2);
System.out.println(num1 + num2);      // 3 (이제 진짜 연산)
```

> 💡 **변수의 초기값**: 정수 `0`, 실수 `0.0`, 문자 `' '`, 문자열 `""` 또는 `null`.

---

## 3. 입력 — Scanner

```java
import java.util.Scanner;
Scanner sc = new Scanner(System.in);
```

`next()`와 `nextLine()`은 공백·엔터 처리 방식이 다르다.

| 메소드 | 공백·엔터 |
|------|------|
| `next()` | 구분점으로 사용 → **담기지 않음** |
| `nextLine()` | 공백·엔터도 **그대로 입력받음** |

### 예제 — 이름·나이 입력

```java
import java.util.Scanner;

String name = "";
int age = 0;
Scanner sc = new Scanner(System.in);

System.out.println("나이를 입력해주세요.");
age = sc.nextInt();

System.out.println("이름을 입력해주세요.");
sc.nextLine();               // ★ 버퍼에 남은 엔터(\n) 비우기
name = sc.nextLine();        // 띄어쓰기 포함 이름 입력
System.out.println(age + "살," + name + "님 환영합니다.");
```

> ⚠️ **`sc.nextLine();` 한 줄의 정체**: 앞서 `nextInt()`로 정수를 입력하면 버퍼에 **엔터(`\n`)가 남는다.** 이걸 비우지 않으면 다음 이름 입력이 씹힌다. (C에서 배운 버퍼 문제와 동일!) 그래서 빈 `nextLine()`으로 엔터를 소진하고, 그 뒤 `nextLine()`으로 실제 이름을 받는다.

> 💡 `age = sc.nextInt();`는 `age = Integer.parseInt(sc.next());`를 편하게 줄인 것이다.

---

## 4. 연산자 우선순위와 결합성

연산자 사이에는 우선순위가 있다.

```
최우선 → 단항 → 산술 → 쉬프트 → 관계 → 논리(비트) → 삼항 → 대입
(높음)                                                    (낮음)
```

> 💡 **결합성**: 같은 우선순위 연산자가 여러 개면 알맞은 방향으로 결합된다. 대개는 상식대로 **왼쪽 → 오른쪽**으로 읽는다.

---

## 5. 비트 연산자

| 연산자 | 이름 | 규칙 |
|:---:|------|------|
| `&` | AND | 둘 다 1이면 1 |
| `\|` | OR | 하나라도 1이면 1 |
| `^` | XOR | 두 비트가 다르면 1 |

```java
System.out.println(10 & 11);   // 1010 & 1011 = 1010 = 10
System.out.println(10 | 11);   // 1010 | 1011 = 1011 = 11
System.out.println(10 ^ 11);   // 1010 ^ 1011 = 0001 = 1
```

![Desktop View](/assets/img/Programming-Language/Java/Cast-Operation-Input/1.png){: width="100%" }

---

## 6. 단항 연산자 `~` (비트 반전)

`~`는 0↔1을 반전한다. 그런데 결과가 예상과 다르게 나온다.

> **핵심**: 컴퓨터에는 뺄셈이 없다. `-1`은 **더해서 0이 되는 수**로 표현한다. 최상위 비트가 부호(양수 0, 음수 1)다. 예를 들어 `1111`에 `0001`을 더하면 자리올림이 밖으로 빠져 `0000`이 되므로, **`1111 = -1`**이다.

```java
// ~2 는? 2 = 0010, 반전하면 1101
// 0010 + 1110 = 0 이므로 1110 = -2
// → ~2 = -3? 실제로는 아래 공식으로:
```

정리하면 다음 공식이 성립한다.

```
~a = -a - 1 = -(a + 1)
```

예: `~2 = -(2+1) = -3`, `~0 = -1`.

![Desktop View](/assets/img/Programming-Language/Java/Cast-Operation-Input/1.png){: width="100%" }

---

## 7. 쉬프트 연산자 (`<<`, `>>`)

| 연산자 | 동작 |
|:---:|------|
| `A << B` | A를 B칸 왼쪽으로 비트 이동 |
| `A >> B` | A를 B칸 오른쪽으로 비트 이동 |

> 💡 쉬프트는 **빠른 연산**을 위해 쓴다. 비트 연산이 사칙연산보다 훨씬 빠르기 때문이다. (`<<n`은 `×2^n`, `>>n`은 `÷2^n` 효과)

---

## 📝 정리

```
형변환 · 입력 · 연산
├─ 문자+정수=연산 / 문자열+값=연결
├─ 변환    Integer.parseInt / Double.parseDouble
├─ 입력    Scanner, next() vs nextLine() (버퍼 주의)
├─ 우선순위 단항>산술>쉬프트>관계>논리>삼항>대입
├─ 비트    & | ^  (AND/OR/XOR)
├─ ~       ~a = -(a+1)
└─ 쉬프트  << >> (빠른 연산)
```

| 개념 | 한 줄 정의 |
|------|------|
| **parseInt** | 문자열을 정수로 변환 |
| **nextLine()** | 공백·엔터 포함 한 줄 입력 |
| **버퍼 비우기** | nextInt 후 남은 엔터를 nextLine으로 소진 |
| **~a = -(a+1)** | 비트 반전의 정수 공식 |

문자열↔숫자 변환과 Scanner의 버퍼 문제는 입문 단계에서 가장 자주 막히는 지점이다. `~a = -(a+1)` 공식처럼, 결과를 외우기보다 **왜 그런지**를 이해하면 오래 남는다.
