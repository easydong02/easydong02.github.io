---
title: "[Java] Optional"
date: 2024-01-11 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, optional]
render_with_liquid: false
---

# Optional

## ✅Java의 Optional: 값이 없는 상황을 편하게 다루자!

---

Java에서 `Optional` 클래스는 값이 존재할 수도, 없을 수도 있는 상황을 처리하는 유용한 도구입니다. 이 클래스는 코드를 깔끔하게 유지하면서 null 체크를 간소화하고 예외 처리를 더 효율적으로 할 수 있게 해줍니다. 그럼 함께 `Optional` 클래스에 대해 알아보도록 하겠습니다.

## ✅Optional이란 무엇인가?

`Optional`은 값이 존재할 수도, 없을 수도 있는 컨테이너입니다. 말 그대로 '값이 있을지도 모르고 없을지도 모른다'는 상황에서 사용됩니다. 이를 통해 null을 직접 다루는 것보다 더 명시적으로 상황을 다룰 수 있습니다.

## 📌기본 사용법

---

### 값이 있는 경우

```java
Optional<String> optionalValue = Optional.of("Hello, Optional!");
String result = optionalValue.orElse("Default Value");
System.out.println(result); // 출력: Hello, Optional!

```

### 값이 없는 경우

```java
Optional<String> emptyOptional = Optional.empty();
String result = emptyOptional.orElse("Default Value");
System.out.println(result); // 출력: Default Value

```

## ✅메소드 체이닝

---

`Optional`은 메소드 체이닝을 통해 여러 작업을 연속적으로 수행할 수 있습니다.

```java
String result = optionalValue
    .map(value -> value + " - Modified")
    .orElse("Default Value");
System.out.println(result); // 출력: Hello, Optional! - Modified

```

## ✅예외 처리 간소화

---

기존에는 null 체크를 통해 예외 처리를 해야 했습니다. 그러나 `Optional`을 사용하면 간단하게 처리할 수 있습니다.

```java
public String getUserName(User user) {
    return Optional.ofNullable(user)
        .map(User::getName)
        .orElse("Unknown");
}

```

## ✅주의할 점

---

`Optional`은 무분별한 사용보다는 상황에 맞게 사용해야 합니다. 과도한 사용은 코드를 복잡하게 만들 수 있으므로 신중하게 선택해야 합니다.