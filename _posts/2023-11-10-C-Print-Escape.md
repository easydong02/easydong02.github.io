---
title: "[C] 기본출력과 Escape문자"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **`printf`를 이용한 문자열 출력**과, 문자열 안에서 특수한 기능을 하는 **escape 문자**를 정리한다.

> `printf`의 뜻은 **print formatted(서식화된 출력)**다. 쌍따옴표 `"..."` 안에는 출력할 **문자열(string)**이 들어간다.

---

## 1. 문자열 선언과 출력

문자열을 선언하는 방법은 여러 가지다. 예시로 세 가지를 보자.

### ① 크기를 지정해서 선언

최대 20개의 문자를 담을 수 있는 문자열 `str1`을 선언한다. (공백도 하나의 문자이므로 `C Language`는 총 10문자)

```c
char str1[20] = "C Language";
printf("str1=%s\n", str1);
```

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/1.png){: width="100%" }

### ② 크기를 비워서 선언 (자동 초기화)

크기를 비워두면 **문자열의 문자 개수만큼 자동으로 크기가 정해진다.**

```c
char str2[] = "Java Language";
printf("str2=%s\n", str2);
```

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/2.png){: width="100%" }

### ③ 포인터로 선언

`*`는 포인터를 사용한 것이다. (자세한 내용은 포인터 편에서 다룬다.)

```c
char *str3 = "Python Language";
printf("str3=%s\n", str3);
```

| 선언 방식 | 예시 | 특징 |
|------|------|------|
| 크기 지정 | `char str1[20]` | 최대 20문자 저장 공간 확보 |
| 크기 생략 | `char str2[]` | 문자 개수만큼 크기 자동 결정 |
| 포인터 | `char *str3` | 문자열 상수를 가리킴 |

---

## 2. Escape 문자

`\n`처럼 줄바꿈에 쓰는 문자도 escape 문자다. **문자열 안(`"..."` 안)에서 특수한 기능을 하는 문자**다.

| Escape 문자 | 기능 |
|:---:|------|
| `\n` | 줄바꿈 |
| `\t` | 탭 (8칸을 한 탭) |
| `\"` | 쌍따옴표 출력 |
| `\\` | 백슬래시(`\`) 출력 |
| `\0` | 널(null) 문자 |

### 활용 예 — 쌍따옴표 출력하기

`say "hi"`라는 문구를 출력하고 싶다. 그런데 쌍따옴표는 문자열의 시작/끝을 나타내는 특수 기호라 그냥 쓸 수 없다. 그래서 escape 문자 `\"`를 사용한다.

```c
printf("say \"hi\"\n");
```

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/3.png){: width="100%" }

원하는 대로 `say "hi"`가 잘 출력됐다.

---

## 📝 정리

| 개념 | 한 줄 정의 |
|------|------|
| **`printf`** | print formatted, 서식화된 출력 |
| **`%s`** | 문자열을 출력하는 서식 지정자 |
| **문자열 선언** | 크기 지정 / 크기 생략(자동) / 포인터 |
| **escape 문자** | 문자열 안에서 특수 기능을 하는 문자 (`\n`, `\t`, `\"` 등) |

문자열 안에서 특수 기호(`"`, `\`)를 직접 출력해야 할 때는 escape 문자를 떠올리자. 이번 시간은 여기까지!
