---
title: "[Spring] @Bean과 static 주의점"
date: 2024-02-20 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, bean, static, singleton]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **`@Configuration` + `@Bean`**에서 **static 메소드**를 쓸 때 주의할 점을 정리한다.

> `@Configuration`과 `@Bean`은 스프링의 **IoC 컨테이너**에 빈의 생성·관리를 위임하는 어노테이션이다. 그런데 여기에 `static`을 붙이면 문제가 생긴다.

---

## 1. static 빈 정의 — 문제 상황

```java
@Configuration
public class AppConfig {
    @Bean
    public static MyBean myBean() {   // ⚠️ static
        return new MyBean();
    }
}
```

> ⚠️ static 메소드로 정의된 빈은 외부에서 직접 호출 가능해, **스프링이 관리하는 대신 개발자가 직접 생성**하는 상황이 될 수 있다.

**문제점 2가지:**

| 문제 | 설명 |
|------|------|
| **단위 테스트 어려움** | static 메소드는 Mock 처리·직접 호출이 어려워 테스트가 복잡 |
| **상태 공유 문제** | static은 모든 곳에서 공유되어 멀티스레드 환경에서 상태 공유 문제 유발 |

---

## 2. 해결 — 인스턴스 메소드로

```java
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean() {   // ✅ 인스턴스 메소드
        return new MyBean();
    }
}
```

> 💡 인스턴스 메소드로 정의하면 **스프링 컨테이너가 빈의 인스턴스를 생성·관리**한다. 그래서 **테스트가 쉽고 상태 공유 문제도 해결**된다.

| 방식 | 관리 주체 | 테스트 |
|------|:---:|:---:|
| `static @Bean` | 개발자(직접) | 어려움 |
| **인스턴스 `@Bean`** | **스프링 컨테이너** | 쉬움 |

---

## 📝 정리

```
@Configuration + @Bean
├─ static 문제  테스트 어려움 + 상태 공유
└─ 해결         인스턴스 메소드로 (컨테이너가 관리)
```

| 개념 | 한 줄 정의 |
|------|------|
| **@Bean** | 스프링 빈을 정의 |
| **static @Bean** | 지양 (관리·테스트 문제) |
| **인스턴스 @Bean** | 권장 (컨테이너 관리) |

`@Bean`을 정의할 때는 **static을 피하고 인스턴스 메소드**를 쓰자. 그래야 스프링 컨테이너가 빈을 제대로 관리하고, 테스트도 쉬워지며 상태 공유 문제도 방지된다.
