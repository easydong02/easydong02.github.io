---
title: "[C] 조건문"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

프로그램은 항상 위에서 아래로만 흐르지 않는다. 상황에 따라 다른 길로 가야 할 때가 있고, 그 흐름을 바꾸는 것이 **제어문(Control)**이다.

> **제어문**: 프로그램의 흐름을 제어(변경)하는 문장.

| 종류 | 문법 |
|------|------|
| **조건문(Conditional)** | `if ~ else`, `switch ~ case` |
| **순환문(Loop)** | `for`, `while`, `do ~ while` |

이번 글에서는 이 중 **조건문**을 다룬다.

---

## 1. if ~ else 조건문

`if(조건식)`은 **조건식이 '참'이면** 그다음의 '한 문장' 또는 '한 블록'을 실행하고, **거짓이면 실행하지 않고** 넘어간다.

![Desktop View](/assets/img/Programming-Language/C/Condition/1.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/2.png){: width="100%" }

`if`만 쓰면 심심하다. 여기에 `else if`와 `else`를 더한다.

```c
if (조건식1)         // 조건식1이 참일 때 수행
    ...
else if (조건식2)    // 위가 거짓이고 조건식2가 참일 때 수행
    ...
else if (조건식3)    // 위가 모두 거짓이고 조건식3이 참일 때 수행
    ...
else                 // 위 조건이 모두 거짓일 때 수행
    ...
```

| 키워드 | 실행 조건 |
|------|------|
| `if` | 조건식이 참일 때 |
| `else if` | 앞선 조건이 거짓이고, 자신의 조건이 참일 때 |
| `else` | 앞선 모든 조건이 거짓일 때 (마지막 처리) |

> 💡 여러 분기 중 **하나가 실행되면 나머지는 실행되지 않는다.** `else if`가 실행되면 그 아래 문장들은 건너뛴다.

![Desktop View](/assets/img/Programming-Language/C/Condition/3.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/4.png){: width="100%" }

`point`에 어떤 값을 넣느냐에 따라 뒤의 조건식 중 하나가 실행된다. 어느 조건에도 해당하지 않으면 마지막 `else`가 실행된다.

---

## 2. switch 조건문

`switch`는 **하나의 정수 값**을 여러 `case`와 비교해 분기한다.

```c
switch (정수)
{
    case x:          // 괄호 안의 값이 x일 때 실행
        ...
        break;       // switch 블록에서 빠져나옴
}
```

- `case` 오른쪽의 값은 `switch(정수)` 괄호 값과 비교할 후보다.
- 각 `case`마다 **`break;`**를 넣어 실행을 멈추고 `switch` 블록을 빠져나온다.

![Desktop View](/assets/img/Programming-Language/C/Condition/5.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/6.png){: width="100%" }

`n`의 값을 1로 설정했기 때문에 `case 1`이 실행되었다.

![Desktop View](/assets/img/Programming-Language/C/Condition/7.png){: width="100%" }

> 💡 `switch` 괄호 안에는 하나의 **식(expression)**을 넣어도 된다. 단, 그 결과값은 반드시 **정수**여야 한다.

---

## 3. 중첩(nested) 조건문

**중첩 조건문**은 말 그대로 **조건문 안에 있는 조건문**이다.

![Desktop View](/assets/img/Programming-Language/C/Condition/8.png){: width="100%" }

```
if (n % 2 == 0)         // 짝수인가?
    if (n % 3 == 0)     //   짝수이면서 3의 배수 → 안쪽 if
        ...
    else                //   짝수지만 3의 배수 아님 → 안쪽 else
        ...
else                    // 홀수
    ...
```

`n`이 짝수인지 먼저 판단하고, 짝수라면 다시 그 안에서 3의 배수인지 판단한다.

![Desktop View](/assets/img/Programming-Language/C/Condition/9.png){: width="100%" }

`6`을 입력하면 (짝수이자 3의 배수이므로) 바깥 `if`와 안쪽 `if`가 모두 실행된다.

---

## ⚠️ 주의사항

**1. 조건문 뒤에 세미콜론(`;`)을 붙이지 말 것**

```c
if (10 < 4);   // ❌ 여기서 if 문장이 끝나버린다!
{
    // 이 블록은 if와 무관하게 '무조건' 실행됨
}
```

`if` 바로 뒤에 `;`을 붙이면 거기서 `if`가 끝난다. 그러면 뒤의 블록은 조건과 상관없이 **무조건 실행**되므로 주의해야 한다.

**2. 비교연산자 `==`와 대입연산자 `=`를 헷갈리지 말 것**

수학에서 쓰던 등호(`=`)는 C에서 **대입연산자**다. "같은가?"를 비교할 때는 반드시 **`==`**를 써야 한다. (부등호 `<`, `>`는 수학과 동일하게 사용)

---

## 📝 정리

```
조건문
├─ if ~ else if ~ else   조건이 참인 분기 하나만 실행
├─ switch ~ case         정수 값을 case와 비교, break로 탈출
└─ 중첩 조건문           조건문 안에 조건문

⚠️ if 뒤 세미콜론 금지  /  비교는 == , 대입은 =
```

조건문은 프로그램에 '판단'을 부여한다. 특히 `==`와 `=`, 그리고 `if` 뒤 세미콜론 두 가지 함정만 조심하면 실수를 크게 줄일 수 있다.
