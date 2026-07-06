---
title: "[Java] Optional"
date: 2024-01-11 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, optional]
render_with_liquid: false
---

## 📌 들어가며

Java를 쓰다 보면 가장 자주 만나는 예외가 `NullPointerException`이다. 값이 있을 수도, 없을 수도 있는 상황을 `null`로 직접 다루면 곳곳에 `if (x != null)` 체크가 흩어진다. 이 문제를 우아하게 다루도록 도와주는 것이 **`Optional`**이다.

> **`Optional`이란?**
> 값이 존재할 수도, 없을 수도 있는 **컨테이너**. "값이 있을지도 없을지도 모른다"는 상황을 `null`보다 **명시적으로** 표현하게 해준다.

---

## 1. 기본 사용법

`Optional`은 값의 유무에 따라 다르게 동작한다. `orElse`로 값이 없을 때의 **기본값**을 지정할 수 있다.

| 상황 | 생성 | 결과 |
|------|------|------|
| 값이 있음 | `Optional.of("Hello")` | 원래 값 반환 |
| 값이 없음 | `Optional.empty()` | 기본값 반환 |

```java
// 값이 있는 경우
Optional<String> optionalValue = Optional.of("Hello, Optional!");
String result = optionalValue.orElse("Default Value");
System.out.println(result);   // 출력: Hello, Optional!

// 값이 없는 경우
Optional<String> emptyOptional = Optional.empty();
String result2 = emptyOptional.orElse("Default Value");
System.out.println(result2);  // 출력: Default Value
```

---

## 2. 메소드 체이닝

`Optional`은 **메소드 체이닝**으로 여러 작업을 연속 수행할 수 있다. `map`으로 값을 변환하고, `orElse`로 마무리한다.

```java
String result = optionalValue
    .map(value -> value + " - Modified")
    .orElse("Default Value");
System.out.println(result);   // 출력: Hello, Optional! - Modified
```

---

## 3. 예외 처리 간소화

기존에는 `null` 체크로 방어 코드를 작성해야 했다. `Optional`을 쓰면 이 과정이 한 줄로 깔끔해진다.

```java
public String getUserName(User user) {
    return Optional.ofNullable(user)   // user가 null이어도 안전
        .map(User::getName)            // null이면 map은 건너뜀
        .orElse("Unknown");            // 최종적으로 없으면 기본값
}
```

| 메소드 | 역할 |
|------|------|
| `Optional.of(x)` | `x`를 감싼 Optional (x가 null이면 예외) |
| `Optional.ofNullable(x)` | `x`가 null이면 empty, 아니면 감쌈 |
| `Optional.empty()` | 빈 Optional |
| `map(f)` | 값이 있으면 변환, 없으면 그대로 empty |
| `orElse(기본값)` | 값이 없을 때 반환할 기본값 |

---

## ⚠️ 주의할 점

> `Optional`은 **무분별하게 쓰기보다 상황에 맞게** 써야 한다. 과도하게 사용하면 오히려 코드가 복잡해지므로 신중하게 선택하자. (예: 필드 타입이나 메소드 파라미터로 남발하는 것은 권장되지 않는다.)

---

## 📝 정리

| 개념 | 한 줄 정의 |
|------|------|
| **Optional** | 값의 유무를 명시적으로 표현하는 컨테이너 |
| **orElse** | 값이 없을 때의 기본값 |
| **map 체이닝** | null 체크 없이 값 변환을 연속 수행 |
| **ofNullable** | null 가능성이 있는 값을 안전하게 감싸기 |

`Optional`은 `null`을 직접 다루는 대신 **"값이 없을 수 있음"을 타입으로 드러내는** 도구다. 덕분에 `NullPointerException`을 줄이고 코드 의도를 명확하게 표현할 수 있다.
