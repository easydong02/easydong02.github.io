---
title: "[C] 변수의 선언과 서식문자열"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

프로그램은 결국 데이터를 다루는 일이고, 그 데이터를 담는 그릇이 **변수(variable)**다. 이번 글에서는 변수의 **선언 → 초기화 → 사용** 흐름과, 출력에 쓰이는 **서식 지정자(`%d`)**를 정리한다.

> **변수란?**
> 프로그램에서 데이터를 담기 위한 공간. 이 공간에 **이름(변수명, variable name)**을 붙여 관리한다.

---

## 1. 변수의 선언

변수는 **사용하기 전에 반드시 선언(declaration)**해야 한다. 선언 형식은 다음과 같다.

```c
[변수타입] [변수명];

int Number;   // 정수 타입의 변수 Number 선언
```

앞의 `int`는 **정수 타입**의 변수를 의미한다. (타입은 뒤에서 자세히 다룬다.)

변수는 보통 다음 세 단계로 이루어진다.

```
선언(declaration) ──► 초기화(initialization) ──► 사용
   int Number;          Number = 10;             printf(...)
                        (최초값 대입 = assign)
```

- **초기화(initialization)**: 변수에 **최초값을 대입(assign)**하는 것.

---

## 2. 초기화하고 출력해보기

정수 타입 변수에 10을 넣고, 제대로 초기화됐는지 출력해 확인한다.

```c
int Number = 10;
printf("Number=%d\n", Number);
```

![Desktop View](/assets/img/Programming-Language/C/Def-Variables-Format-String/1.png){: width="100%" }

여기서 왜 서식 문자열에 **`%d`**를 썼을까? `%d`는 **서식 지정자(format specifier)** 중 정수 타입을 위한 것이다. 변수 `Number`를 정수 타입으로 선언했기 때문이다. 즉, **`%d` 자리에 `Number`의 값을 넣어 출력**한다는 뜻이다.

![Desktop View](/assets/img/Programming-Language/C/Def-Variables-Format-String/2.png){: width="100%" }

정상적으로 `10`이 출력됐다.

---

## 3. 변수 사용 시 주의사항

> ⚠️ 다음 규칙을 어기면 컴파일 오류나 예기치 않은 동작이 발생한다.

| 규칙 | 설명 |
|------|------|
| **선언 후 사용** | 코드는 위→아래, 왼쪽→오른쪽으로 읽히므로 반드시 선언 뒤에 사용 |
| **숫자로 시작 금지** | 변수명은 숫자로 시작할 수 없음 |
| **띄어쓰기 금지** | 변수명 중간에 공백 불가 |
| **대소문자 구분** | `Number`와 `number`는 다른 변수 |
| **초기화 후 사용** | 초기값이 대입되지 않은 변수는 사용 불가 |

---

## 📝 정리

```
변수(variable)
├─ 선언    int Number;        타입 + 이름
├─ 초기화  Number = 10;       최초값 대입(assign)
├─ 사용    printf("%d", ...); 서식 지정자로 값 출력
└─ 규칙    선언 후 사용 / 숫자 시작·공백 불가 / 대소문자 구분
```

서식 문자열에 대한 더 자세한 내용은 다음 글에서 이어서 정리하겠다.
