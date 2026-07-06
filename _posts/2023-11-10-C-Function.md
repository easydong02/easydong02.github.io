---
title: "[C] 함수"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

같은 코드를 여기저기 반복해서 쓰는 건 비효율적이다. 자주 쓰는 코드 덩어리를 하나로 묶어 이름을 붙여두고, 필요할 때마다 불러 쓰는 것이 **함수(function)**다.

> **함수란?**
> 자주 반복 수행할 코드 덩어리를 ① **함수로 '정의'**하고, ② 필요할 때마다 **'호출'**하여 사용하는 것.

---

## 1. 함수의 정의·호출·리턴

### 함수 정의 (function definition)

```c
리턴타입 함수이름(매개변수들 ...)
{
    함수 본체 (수행 코드들)
}
```

- **함수 원형(function prototype)** = `리턴타입 함수이름(매개변수들)`. 함수를 **선언만** 할 때 이 원형을 쓴다.

![Desktop View](/assets/img/Programming-Language/C/Function/1.png){: width="100%" }

> 💡 Visual Studio 소스 파일 왼쪽의 삼각형 버튼을 열고 닫으면, 그 파일에 어떤 함수가 원형 형태로 쓰였는지 볼 수 있다.

### 함수 호출 (function call)

```c
함수이름(매개변수에 넣을 값);
```

우리가 늘 쓰던 `main()`도 사실 함수의 일종이다.

### 함수의 실행 흐름

컴파일러가 **호출 코드**를 만나면 잠깐 멈추고 **호출된 함수**를 실행한다. 다 읽으면 멈췄던 곳으로 **돌아온다(return)**.

```
main() 실행 중  ──호출──►  oper() 실행
     ▲                        │
     └────── return ──────────┘
        (멈췄던 지점부터 계속)
```

> TMI: 호출한 함수를 **caller**, 호출된 함수를 **callee**라고 한다.

### 매개변수 vs 인자

| 용어 | 정의 |
|------|------|
| **매개변수(parameter)** | 함수를 **정의할 때** 명시하는 변수 (그 함수의 지역변수로 동작) |
| **인자(argument)** | 함수를 **호출할 때** 전달하는 값 |

함수 호출 시 **인자가 매개변수로 '복사'**되어 실행된다. (두 용어는 실무에서 혼용되기도 한다.)

---

## 2. 예제로 보는 함수

`oper` 함수를 만들어 보자. 매개변수는 `int` 2개, 리턴타입은 `void`(리턴값 없음)다. 본체에서 사칙연산을 수행한다.

![Desktop View](/assets/img/Programming-Language/C/Function/2.png){: width="100%" }

`main` 밖에 함수를 만들었다. 그런데 이대로 실행하면 `oper` 안의 문장이 실행될까? **아니다.** `main`에서 **호출**해야 한다.

![Desktop View](/assets/img/Programming-Language/C/Function/3.png){: width="100%" }

`oper(100, 10)`은 "k에는 100, v에는 10을 넣어 실행"한다는 뜻이다.

![Desktop View](/assets/img/Programming-Language/C/Function/4.png){: width="100%" }

> ⚠️ 실무에서는 함수들이 서로를 호출하며 복잡하게 얽힌다. `main`이 `aaa()`를 호출하고, `aaa()` 안에서 다시 `bbb()`를 호출하는 식이다. 따라서 **함수의 논리적인 호출 순서**를 잘 설계해야 한다.

---

## 3. 리턴(return)

> **리턴값**: 자기를 호출한 함수에게 돌려주는 **최종값 1개**. 매개변수로 식을 수행한 뒤 나온 결과값이다.

![Desktop View](/assets/img/Programming-Language/C/Function/5.png){: width="100%" }

두 가지 방식을 비교하자.

```c
// ① 변수에 담아 리턴
int add(int a, int b) {
    int sum = a + b;
    return sum;
}
int result = add(10, 5);   // result = 15

// ② 식을 그대로 리턴
int sub(int a, int b) {
    return a - b;          // 결과는 동일
}
```

- `add(10, 5)`의 리턴값은 15이므로, `int result = add(10, 5);`에서 `result`에 15가 대입된다.
- `void` 타입은 리턴값이 없으니, `printf` 같은 문장으로 결과를 처리하면 된다.

---

## 4. 순서 — '선언'과 '정의'

변수를 선언 후에 쓰듯, 함수도 **정의(또는 최소한 선언)가 호출보다 앞**에 있어야 한다. 컴파일러가 위→아래로 읽기 때문이다.

![Desktop View](/assets/img/Programming-Language/C/Function/6.png){: width="100%" }

`main` 아래에 `half` 함수를 정의하면 오류가 난다. 하지만 방법이 있다. **'정의'는 아래에서 하되, '선언'만 위에서** 해주면 된다.

```c
double half(int n);   // 위에서 '선언' (함수 원형)

int main() {
    ... half(10) ...   // 호출 가능!
}

double half(int n) {   // 아래에서 '정의'
    return n / 2.0;
}
```

![Desktop View](/assets/img/Programming-Language/C/Function/7.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Function/8.png){: width="100%" }

잘 실행된다.

---

## 5. 헤더 파일로 선언 분리하기

여러 소스 파일마다 함수를 일일이 선언·정의하는 건 번거롭다. 그래서 **헤더 파일(`.h`)**에 함수의 **선언**을 모아둔다.

![Desktop View](/assets/img/Programming-Language/C/Function/9.png){: width="100%" }

> **헤더 파일이란?** 함수의 선언, 매크로, 상수 등을 모아둔 파일. 우리가 늘 쓰는 `#include <stdio.h>`도 헤더 파일이다. 기본 함수들의 선언이 거기 있어서, 그걸 포함해야 그 함수들을 쓸 수 있다.

| 구분 | 포함 방식 | 예 |
|------|------|------|
| **표준 헤더** | `#include <...>` | `#include <stdio.h>` |
| **직접 만든 헤더** | `#include "..."` | `#include "myfunction.h"` |

**역할 분리**를 그림으로 정리하면 이렇다.

```
myfunction.h  ←  함수 '선언'만 모음
myfunction.c  ←  함수 '정의'만 모음
test.c(main)  ←  #include "myfunction.h" 후 '호출'만
```

![Desktop View](/assets/img/Programming-Language/C/Function/10.png){: width="100%" }

프로젝트의 소스 파일 목록을 보면, 빌드에서 제외(금지 표시)된 파일들이 있다. 활성화된 것은 **① `main`을 포함한 파일**과 **② 함수 정의를 모은 파일** 두 개다. (정의 파일에는 `main`이 없다.)

![Desktop View](/assets/img/Programming-Language/C/Function/11.png){: width="100%" }

정리하면 이렇다.

> **한 소스 파일**에 함수를 **'정의'**하고, **헤더 파일**에 함수를 **'선언'**하고, `main`이 있는 소스 파일에 그 헤더를 **`#include`**하면 → `main` 파일에서는 그냥 **'호출'만** 하면 된다.

`power(3, 4)` (3의 4제곱 = 81)를 예로 확인해 보자. 정의는 `myfunction.c`, 선언은 헤더에, 호출은 `test.c`에서.

![Desktop View](/assets/img/Programming-Language/C/Function/12.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Function/13.png){: width="100%" }

`main`에 선언도 정의도 없이 **호출만으로** 잘 동작한다.

---

## 6. 재귀 호출 (recursive call)

> **재귀 호출**: 함수가 실행 중에 **자기 자신을 다시 호출**하는 것.

| 특징 | 설명 |
|------|------|
| 장점 | 복잡한 문제를 단순화하여 구현 가능 |
| 제약 | **무한 재귀는 불가** (반드시 종료 조건 필요) |

![Desktop View](/assets/img/Programming-Language/C/Function/14.png){: width="100%" }

`func1`의 리턴값이 또 `func1(n+1)`이다. `main`에서 `func1(1)`을 호출하면 어떻게 될까? **무한히** 호출된다.

![Desktop View](/assets/img/Programming-Language/C/Function/15.png){: width="100%" }

> ⚠️ 그래서 함수 안에 `if (n == ?)` 같은 **종료 조건(base case)**을 두고, 그때의 리턴값을 지정해 무한 호출을 막아야 한다.

---

## 📝 정리

```
함수(function)
├─ 정의    리턴타입 이름(매개변수) { 본체 }
├─ 호출    이름(인자);  → caller가 멈추고 callee 실행 후 return
├─ 매개변수/인자  정의 시 변수 / 호출 시 전달값 (복사됨)
├─ return  호출자에게 최종값 1개 반환
├─ 순서    선언(원형)만 위에 있으면 정의는 아래여도 OK
├─ 헤더    .h에 '선언' 모음 → #include 로 재사용
└─ 재귀    자기 자신 호출, 반드시 종료 조건 필요
```

| 개념 | 한 줄 정의 |
|------|------|
| **함수** | 반복 코드를 정의하고 호출해 재사용하는 단위 |
| **원형(prototype)** | `리턴타입 이름(매개변수)`, 선언에 사용 |
| **매개변수/인자** | 정의 시 변수 / 호출 시 전달값 |
| **헤더 파일** | 함수 선언·매크로·상수를 모아 재사용 |
| **재귀 호출** | 자기 자신을 호출, 종료 조건 필수 |

함수는 프로그래밍의 '효율과 재사용'을 담당하는 핵심 도구다. 선언과 정의의 분리, 그리고 헤더 파일로 이어지는 흐름을 이해하면 규모가 큰 프로젝트도 깔끔하게 구성할 수 있다.
