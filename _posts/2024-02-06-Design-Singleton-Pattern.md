---
title: "[DesignPattern] 싱글턴 패턴(Singleton Pattern)"
date: 2024-02-06 00:00:00 +0900
categories: [Design-Pattern]
tags: [designpattern, singletonpattern]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 객체지향 설계 패턴 중 자주 쓰이는 **싱글턴 패턴(Singleton Pattern)**을 정리한다.

> **싱글턴 패턴이란?**
> 특정 클래스의 인스턴스가 **오직 하나만** 생성되도록 보장하고, 그 인스턴스에 대한 **전역 접근점**을 제공하는 패턴.

| 특징 | 설명 |
|------|------|
| **유일한 인스턴스** | 클래스의 인스턴스가 하나만 존재 |
| **전역 접근성** | 어디서든 그 인스턴스에 접근 |

> 💡 주로 **공유 자원 접근 제어**, 설정 관리자·로그·캐시 관리자 등에 쓰인다.

---

## 1. 구현 방법 2가지

### ① 지연 초기화 (Lazy)

```java
public class Singleton {
    private static Singleton instance;
    private Singleton() {}   // private 생성자로 외부 생성 차단

    public static Singleton getInstance() {
        if (instance == null) {          // 처음 호출 시 생성
            instance = new Singleton();
        }
        return instance;
    }
}
```

### ② 즉시 초기화 (Eager)

```java
public class Singleton {
    private static final Singleton instance = new Singleton();
    private Singleton() {}

    public static Singleton getInstance() {
        return instance;   // 이미 생성된 것 반환
    }
}
```

| 방식 | 생성 시점 | 스레드 안전 |
|------|:---:|:---:|
| **Lazy** | 처음 사용 시 | ❌ (동기화 필요) |
| **Eager** | 클래스 로딩 시 | ✅ |

> 💡 두 방식 모두 **`private` 생성자**로 외부에서 `new`를 못 하게 막고, `getInstance()`로만 인스턴스를 얻게 하는 것이 핵심이다.

---

## 2. 사용 예시

```java
public class Main {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();   // 항상 같은 인스턴스
        singleton.showMessage();   // Hello, Singleton!
    }
}
```

---

## 📌 주의사항

| 항목 | 내용 |
|------|------|
| **멀티스레드** | Lazy 방식은 동기화 처리 필요 |
| **리플렉션** | 리플렉션을 통한 객체 생성을 막아야 함 |
| **직렬화** | 역직렬화 시 새 인스턴스 생성 방지 |

---

## 📝 정리

```
싱글턴 패턴
├─ 목적    인스턴스 하나 + 전역 접근
├─ 핵심    private 생성자 + getInstance()
├─ 방식    Lazy(지연, 동기화 필요) / Eager(즉시, 안전)
└─ 주의    멀티스레드 · 리플렉션 · 직렬화
```

| 개념 | 한 줄 정의 |
|------|------|
| **싱글턴** | 인스턴스가 하나뿐인 패턴 |
| **private 생성자** | 외부 생성 차단 |
| **Lazy vs Eager** | 지연 생성 vs 즉시 생성 |

싱글턴은 "인스턴스는 단 하나"를 보장하는 패턴이다. `private` 생성자와 `getInstance()`가 핵심이며, 멀티스레드 환경에서는 Eager 방식이나 동기화로 안전성을 확보해야 한다. (스프링의 빈도 기본이 싱글턴이다.)
