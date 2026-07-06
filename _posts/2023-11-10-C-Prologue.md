---
title: "[C] Prologue"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

> _The computing world has undergone a revolution since the publication of The C Programming Language in 1978._
>
> (1978년 책 「The C Programming Language」 출판 이후 컴퓨팅 세계는 혁명을 겪어왔다.)
>
> — The C Programming Language 2nd Edition, *Dennis MacAlistair Ritchie*

C 언어 공부를 시작하며 첫 글을 남긴다. 앞으로 이 시리즈에서는 남의 방식이 아니라 **내가 배운 것과 느낀 것**을 중심으로 기록해 나가려 한다.

---

## 1. C 언어란?

**1972년 벨 연구소(Bell Labs)의 데니스 리치(Dennis Ritchie)가 만든** 프로그래밍 언어다. 원래 명칭은 그냥 'C'지만, 한국에서는 표제어에서 보이듯 'C언어'라고 주로 부른다. 세계적으로 **가장 많이 쓰이는 프로그래밍 언어** 중 하나다.

| 항목 | 내용 |
|------|------|
| **만든 사람** | 데니스 리치 (Dennis Ritchie) |
| **탄생** | 1972년, 벨 연구소 |
| **위상** | 세계에서 가장 널리 쓰이는 언어 중 하나 |
| **성격** | 저수준 제어가 가능한 절차적 언어 |

---

## 2. C의 정신 — "프로그래머를 믿어라"

C의 설계 철학은 **C99 Rationale**에 다음과 같이 묘사되어 있다.

| 원칙 (원문) | 의미 |
|------|------|
| **Trust the programmer** | **프로그래머를 믿어라** |
| Don't prevent the programmer from doing what needs to be done | 프로그래머가 할 일을 방해하지 마라 |
| Keep the language small and simple | 언어를 작고 간단하게 유지하라 |
| Provide only one way to do an operation | 한 작업을 하는 방법은 하나만 제공하라 |
| Make it fast, even if it is not guaranteed to be portable | 이식성을 장담 못 해도, 빠르게 만들어라 |

C를 다른 언어들과 가장 크게 구분 짓는 것은 첫 줄, **"프로그래머를 믿어라"**다.

```
[현대 언어들]                        [C 언어]
프로그래머를 100% 믿지 않는다          프로그래머를 믿는다
      │                                   │
문제가 생길 것 같으면                   "하라는 대로 한다"
컴파일러·가상머신이 자동으로            → 자동 첨삭 없이
픽싱하거나 비정상 코드를 방지           본연 그대로의 프로그래밍
```

> 💡 비교적 현대의 언어들은 프로그래머를 완전히 믿지 않아서, 문제가 될 것 같으면 컴파일러나 가상머신이 자동으로 고쳐주거나 막아준다. 반면 C는 **"하라는 대로 한다"**는 특징이 매력적이다. 언어의 자동적인 첨삭이 아닌 **본연 그대로의 프로그래밍**을 배우는 경험은 앞으로의 학습에도 좋은 영향을 줄 것으로 기대된다.

---

## 📝 정리

- C는 1972년 데니스 리치가 만든, 가장 널리 쓰이는 언어 중 하나다.
- 핵심 철학은 **"프로그래머를 믿어라(Trust the programmer)"** — 자동 보정 없이 지시한 그대로 동작한다.
- 이 시리즈는 정답을 옮겨 적는 것이 아니라, 배우고 느낀 것을 내 방식대로 기록하는 데 목적이 있다.
