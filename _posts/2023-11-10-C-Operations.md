---
title: "[C] 연산자"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

연산자에는 여러 종류가 있다. **산술·복합 대입·증감·비교·논리·삼항·비트 연산자** 등. 이번 글에서는 이들을 하나씩 정리한다.

| 분류 | 연산자 |
|------|------|
| 산술 | `+` `-` `*` `/` `%` |
| 복합 대입 | `+=` `-=` `*=` `/=` `%=` |
| 증감 | `++` `--` |
| 비교 | `>` `<` `>=` `<=` `==` `!=` |
| 논리 | `&&` `\|\|` `!` |
| 삼항 | `? :` |
| 비트 | `&` `\|` `^` `~` `<<` `>>` |

---

## 1. 산술 연산자

덧셈·뺄셈·곱셈·나눗셈은 익숙하다. 눈여겨볼 것은 **`%`(나머지 연산자)**다.

```c
5 % 2   // 5를 2로 나눈 나머지 → 1
```

![Desktop View](/assets/img/Programming-Language/C/Operations/1.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/2.png){: width="100%" }

`5 % 2`의 결과로 `1`이 출력된다.

---

## 2. 복합 대입 연산자

대부분의 이항연산자는 복합 대입 형태로 줄여 쓸 수 있다.

```c
int n = 10;
n = n + 3;   // 기존 값에 3을 더함
n += 3;      // ↑ 위와 동일. 자기 자신에게 더해 대입
```

![Desktop View](/assets/img/Programming-Language/C/Operations/3.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/4.png){: width="100%" }

`n`이 0에서 3씩 누적되어 잘 출력됐다.

---

## 3. 증감 연산자 (`++`, `--`)

각각 **1씩 증가, 1씩 감소**시킨다. `n++`이면 기존 `n` 값에 1을 더한다.

![Desktop View](/assets/img/Programming-Language/C/Operations/5.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/6.png){: width="100%" }

주의할 점은 **`n++`과 `++n`이 다르다**는 것이다. 다항 연산식에서 증감이 수행되는 **시점**에 차이가 있다.

| 표기 | 방식 | 시점 |
|:---:|------|------|
| `n++` | postfix(후위) | 기존 연산 **직후** 증감 수행 |
| `++n` | prefix(전위) | 기존 연산 **직전** 증감 수행 |

> ⚠️ 가급적 증감 연산자는 복잡한 연산식 **'안'에서 섞어 쓰는 것을 피하는 게** 좋다. 코드가 헷갈리고 버그의 원인이 된다.

---

## 4. 비교 연산자

결과가 **'거짓'이면 0**, **'참'이면 보통 1**이다. (단, 0이 아닌 모든 숫자는 참으로 취급)

![Desktop View](/assets/img/Programming-Language/C/Operations/7.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/8.png){: width="100%" }

정수 서식 지정자로 논리 결과를 출력하면 `1`과 `0`으로 나온다.

---

## 5. 논리 연산자 (`&&`, `||`, `!`)

단순 숫자 계산이 아니라 **논리의 참/거짓**을 가른다.

| 연산자 | 이름 | 참이 되는 조건 |
|:---:|------|------|
| `&&` | AND (논리곱) | 양쪽이 **둘 다 참**일 때만 |
| `\|\|` | OR (논리합) | 둘 중 **하나만 참**이어도 |
| `!` | NOT | 참↔거짓 반전 |

**진리표:**

```
 A && B                A || B              !A
T && T = 1            T || T = 1          !T = 0
T && F = 0            T || F = 1          !F = 1
F && T = 0            F || T = 1
F && F = 0            F || F = 0
```

![Desktop View](/assets/img/Programming-Language/C/Operations/9.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/10.png){: width="100%" }

`result` 값이 논리 결과에 따라 1과 0으로 출력된다.

**NOT 연산자(`!`)** — `10 == 10`은 참(1)이지만, `!(10 == 10)`은 그 반대인 0을 출력한다.

![Desktop View](/assets/img/Programming-Language/C/Operations/11.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/12.png){: width="100%" }

---

## 6. 삼항 연산자

`if ~ else`를 한 줄로 줄인 형태다.

```c
( 조건식 ) ? (참인 경우 결과) : (거짓인 경우 결과)
```

거짓인 경우 자리에 또 다른 삼항 연산자를 이어 붙여 **연쇄**할 수도 있다.

![Desktop View](/assets/img/Programming-Language/C/Operations/13.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/14.png){: width="100%" }

---

## 7. 비트 연산자

논리 연산과 달리 값을 **bit 단위**로 계산한다. 컴퓨터는 비트를 다루는 게 산술 연산보다 훨씬 빠르다는 장점이 있다.

| 연산자 | 이름 | 동작 |
|:---:|------|------|
| `&` | AND | 두 자릿수가 **둘 다 1**일 때만 1 |
| `\|` | OR | 두 자릿수 중 **하나라도 1**이면 1 |
| `^` | XOR | 두 자릿수가 **다르면 1**, 같으면 0 |
| `~` | NOT | 비트 반전 (0↔1) |
| `<<` `>>` | SHIFT | 비트를 좌/우로 이동 |

### & 연산자

각 자릿수가 둘 다 1일 때만 1. 그래서 결과가 대체로 **작아진다**.

![Desktop View](/assets/img/Programming-Language/C/Operations/15.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/16.png){: width="100%" }

### | 연산자

한 자릿수라도 1이면 1. 그래서 결과가 대체로 **커진다**.

![Desktop View](/assets/img/Programming-Language/C/Operations/17.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/18.png){: width="100%" }

### ^ 연산자 (XOR, 배타적 논리합)

둘이 **같으면 0, 다르면 1**.

![Desktop View](/assets/img/Programming-Language/C/Operations/19.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/20.png){: width="100%" }

### ~ 연산자 (NOT, 비트 반전)

`1 ↔ 0`으로 반전. 그런데 `~120`이 왜 `-121`이 될까? **용량(비트 수)**을 봐야 한다.

```
십진수 120을 4byte(32bit)로 표현:
  0000 0000 0000 0000 0000 0000 0111 1000   (120)

비트 반전(~) 적용:
  1111 1111 1111 1111 1111 1111 1000 0111   (= -121)
```

맨 앞 비트가 1이 되면서(부호 비트) **오버플로우처럼 음수**가 된 것이다.

![Desktop View](/assets/img/Programming-Language/C/Operations/21.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/22.png){: width="100%" }

### Shift 연산자 (`<<`, `>>`)

비트를 좌/우로 이동시킨다.

```c
num1 << 2;   // 비트를 왼쪽으로 2칸 이동
```

![Desktop View](/assets/img/Programming-Language/C/Operations/23.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Operations/24.png){: width="100%" }

> 💡 `<< n`은 값을 `2^n`배, `>> n`은 `2^n`으로 나눈 것과 같은 효과를 낸다. 곱셈/나눗셈을 빠르게 처리하는 트릭으로도 쓰인다.

---

## 📝 정리

```
연산자
├─ 산술      + - * / %(나머지)
├─ 복합 대입 n += 3  ≡  n = n + 3
├─ 증감      ++ / --   (n++ 후위 · ++n 전위)
├─ 비교      결과가 참=1, 거짓=0
├─ 논리      && (둘 다 참) · || (하나만 참) · ! (반전)
├─ 삼항      조건 ? 참 : 거짓
└─ 비트      & | ^ ~ << >>  (bit 단위, 매우 빠름)
```

이번 시간에는 연산자를 좀 길게 정리해봤다. 특히 비트 연산자는 처음엔 낯설지만, 원리를 알고 나면 정말 재미있는 영역이다.
