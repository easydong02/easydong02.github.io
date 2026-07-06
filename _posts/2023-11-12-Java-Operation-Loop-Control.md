---
title: "[Java] 누적, 증감 연산자 반복문 제어문"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **누적·증감 연산자**와 **반복문(for, while)**, 그리고 반복 제어문(**break, continue**)을 정리한다.

---

## 1. 복합 대입 · 누적 연산자

먼저 흔한 착각 하나를 짚자.

```java
int money = 10000;
money - 500;                // ❌ 그냥 값일 뿐, money는 그대로
System.out.println(money);  // 10000
```

`money - 500;`은 계산만 하고 **어디에도 저장하지 않는다.** 결과를 다시 `money`에 대입해야 한다.

```java
money = money - 500;   // 복합 대입 (다소 김)
money -= 500;          // 누적 연산자 (간결)
```

| 연산자 | 의미 |
|:---:|------|
| `+=` | `data = data + n` |
| `-=` `*=` `/=` | 각각 빼기·곱하기·나누기 후 대입 |

---

## 2. 증감 연산자 (전위 vs 후위)

값에 1을 더하거나 빼는 연산자다. **전위형과 후위형**은 값 자체는 같게 만들지만, **연산 순서**에서 차이가 난다.

```java
++data;   // 전위: 먼저 1 증가시킨 뒤 사용
data++;   // 후위: 먼저 사용한 뒤 1 증가
```

| 표기 | 동작 |
|:---:|------|
| `++data` | 연산 **전에** 증가 |
| `data++` | 연산 **후에** 증가 |

> 💡 실무에서는 보통 **후위형(`data++`)**을 많이 쓰고, 전위형은 특별한 경우가 아니면 잘 쓰지 않는다.

---

## 3. for 문

C와 매우 유사하다.

```java
for (초기식; 조건식; 증감식) { ... }
```

```java
for (int i = 0; i < 10; i++) {
    System.out.println("신동희");   // 이름 10번 출력
}
```

`i`는 0부터 9까지, 조건식(`i < 10`)이 참인 동안 실행되고 10이 되면 빠져나온다. (총 10번)

### 응용 — aBcDeFg… 번갈아 출력

```java
char aa = 'a';   // 소문자
char bb = 'B';   // 대문자
for (int i = 0; i < 26; i++) {
    if (i % 2 == 0) {           // 짝수 → 소문자
        System.out.print(aa);
        aa += 2;                // a → c → e (한 칸 건너)
    } else {                    // 홀수 → 대문자
        System.out.print(bb);
        bb += 2;                // B → D → F
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Operation-Loop-Control/1.png)

`i`가 0→a, 1→B, 2→c, 3→D… 순으로 소문자와 대문자를 번갈아 출력한다. `+= 2`로 한 칸씩 건너뛴다.

---

## 4. while 문

```java
while (조건식) { 반복할 문장 }
```

`for`보다 간단하게 생겼다. **반복 횟수를 모를 때** 유용하다.

```java
// 정답이 나올 때까지 반복하는 퀴즈
while (choice != answer) {
    System.out.println(qMsg);
    choice = sc.nextInt();
    if (choice == answer)              result = "정답";
    else if (choice >= 1 && choice <= 4) result = "오답";
    else                                result = "다시 시도해주세요.";
    System.out.println(result);
}
```

| 반복문 | 언제 쓰나 |
|------|------|
| `for` | 반복 횟수가 명확할 때 |
| `while` | 반복 횟수를 모를 때(조건 만족까지) |

---

## 5. break와 continue

| 키워드 | 동작 |
|------|------|
| **`break`** | 즉시 **가장 가까운 순환문을 탈출** |
| **`continue`** | 아래 코드를 건너뛰고 **다음 반복으로** |

> ⚠️ `break`는 `if`가 아니라 **순환문**을 탈출한다. `if` 안에 있어도 `if`를 감싼 순환문을 빠져나간다.

### break — 4까지만 출력

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i + 1);   // 출력 먼저
    if (i == 3) { break; }       // 4 출력 후 종료
}
```

> 💡 순서가 중요하다. `if`를 출력문 **앞**에 두면 4를 출력하기 전에 break를 만난다. 출력문 앞에 두려면 조건을 `i == 4`로 바꾸면 된다.

### continue — 4만 제외하고 출력

```java
for (int i = 0; i < 10; i++) {
    if (i == 3) { continue; }    // i=3(=4)이면 건너뜀
    System.out.println(i + 1);
}
```

![Desktop View](/assets/img/Programming-Language/Java/Operation-Loop-Control/2.png)

---

## 📝 정리

```
연산자 · 반복문 · 제어
├─ 누적    money -= 500  ≡  money = money - 500
├─ 증감    ++data(전위) / data++(후위) — 실무는 후위 위주
├─ for     횟수가 명확할 때
├─ while   횟수를 모를 때(조건 충족까지)
└─ 제어    break(순환문 탈출) · continue(다음 반복으로)
```

| 개념 | 한 줄 정의 |
|------|------|
| **누적 연산자** | `+=` 등, 자기 자신에 연산 후 대입 |
| **전위/후위** | 증감을 연산 전/후에 수행 |
| **break/continue** | 순환문 탈출 / 다음 반복으로 |

연산자와 반복문은 프로그래밍의 뼈대다. 특히 `break`/`continue`가 순환문을 대상으로 한다는 점, 그리고 조건 위치에 따라 출력 결과가 달라진다는 점을 기억하자. 다음엔 배열로 돌아온다!
