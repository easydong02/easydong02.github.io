---
title: "[C] 표준 입출력 헤더와 출력"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

어떤 프로그램이든 첫 출력을 찍어보는 것으로 시작한다. 이번 글에서는 C 프로그램의 **기본 틀**과 **`printf`를 이용한 출력**을 정리한다.

---

## 1. C 프로그램의 기본 틀

Visual Studio에서 C 프로그램을 작성하려면 기본적인 '틀'을 갖춰야 한다.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/1.png){: width="100%" }

```c
#include <stdio.h>   // ① 표준 입출력 헤더

int main()           // ② 프로그램의 시작점
{                    // ③ 블록 시작 { → 여기서부터 코드를 읽는다
    printf("...");
}                    //    블록 끝 } → main이 끝나면 프로그램 종료
```

| 구성 요소 | 역할 |
|------|------|
| `#include <stdio.h>` | **표준 입출력(standard input/output) 헤더**. `printf` 등을 쓰기 위해 포함 |
| `main()` | **C 프로그램의 시작점(진입점)** |
| `{ }` | **블록**. 컴파일러가 이 안의 코드를 읽어 실행 |

> 💡 컴파일러는 문장을 **위에서 아래로, 왼쪽에서 오른쪽으로** 읽는다. 코드가 복잡해지면 내가 직접 컴파일러가 된 셈 치고 한 줄씩 따라 읽어보는 것이 디버깅에 큰 도움이 된다.

---

## 2. 출력 — `printf`

출력은 보통 **`printf`**로 한다. 기본 형식은 다음과 같다.

```c
printf("출력하고 싶은 내용");
```

두 가지 규칙을 지켜야 한다.

| 규칙 | 설명 |
|------|------|
| **문자열은 쌍따옴표로** | 출력할 내용(문자열)은 반드시 `"..."`로 묶는다 |
| **문장 끝은 세미콜론** | 하나의 문장(statement)은 반드시 `;`로 끝낸다 |

'동희의 IT 일기장'이라는 문구를 출력해 보자.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/2.png){: width="100%" }

쌍따옴표 안에 문자열을 넣었다. 이제 `Ctrl + F5`로 빌드해 출력을 확인한다.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/3.png){: width="100%" }

정상적으로 원하는 문자열이 출력됐다. 그리고 `main()` 함수가 끝나면 프로그램은 종료된다.

---

## 📝 정리

```
C 프로그램 기본 틀
├─ #include <stdio.h>   표준 입출력 헤더
├─ int main() { ... }   시작점(진입점) + 블록
└─ printf("내용");       출력, "쌍따옴표" + 세미콜론 필수
```

앞으로 배울 것이 많지만, 모든 프로그램의 출발점은 결국 이 작은 틀과 첫 출력이다. 하나씩 차근차근 해보자.
