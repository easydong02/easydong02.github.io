---
title: "[Java] 람다식(Lambda)"
date: 2024-02-02 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, lambda]
render_with_liquid: false
---

## 📌 들어가며

**Java 8**에서 도입된 람다식은 자바 프로그래밍 패러다임을 크게 바꾼 기능이다. 장황하던 익명 클래스를 한 줄로 줄여주고, **함수형 프로그래밍**의 표현력을 자바에 들여왔다. 이번 글에서는 람다식의 기본 개념, 활용, 주의사항을 정리한다.

> **람다식이란?**
> **익명 함수**의 형태를 갖는 자바의 표현 방식. 주로 **함수형 인터페이스**(추상 메소드가 딱 하나인 인터페이스)를 구현할 때 사용해, 코드를 간결하게 만든다.

---

## 1. 기본 문법

익명 클래스와 비교하면 람다식의 간결함이 한눈에 보인다.

```java
// 기존의 익명 클래스
Runnable runnable1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello, World!");
    }
};

// 람다식
Runnable runnable2 = () -> System.out.println("Hello, World!");
```

매개변수와 반환값이 있을 때도 마찬가지다. 람다식은 **타입을 추론**하므로 매개변수 타입을 생략할 수 있다.

```java
// 익명 클래스
Comparator<Integer> comparator1 = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
};

// 람다식 (타입 추론)
Comparator<Integer> comparator2 = (o1, o2) -> o1.compareTo(o2);
```

```
익명 클래스 (여러 줄)          람다식 (한 줄)
new Runnable() {              () -> System.out.println("Hi")
    public void run() {  ──►
        ...
    }
}
```

---

## 2. 람다식의 활용

### 함수형 인터페이스와의 결합

람다식은 **추상 메소드가 하나뿐인** 함수형 인터페이스를 구현할 때 빛을 발한다.

```java
@FunctionalInterface
interface MyFunction {
    int calculate(int a, int b);
}

// 하나의 인터페이스로 다양한 동작을 표현
MyFunction add      = (a, b) -> a + b;
MyFunction subtract = (a, b) -> a - b;
```

### 컬렉션 처리

반복문도 간결하게 표현된다.

```java
List<String> languages = Arrays.asList("Java", "Python", "JavaScript", "Kotlin");

// 기존 방식
for (String language : languages) {
    System.out.println(language);
}

// 람다식
languages.forEach(language -> System.out.println(language));
```

---

## ⚠️ 람다식의 주의사항

| 항목 | 주의점 |
|------|------|
| **함수형 인터페이스 여부** | 람다 대상은 추상 메소드가 **하나뿐인** 인터페이스여야 함 |
| **변수의 final 여부** | 외부 변수 사용 시 `final` 또는 `effectively final`이어야 함 |
| **`this` 키워드** | 람다 내부의 `this`는 **람다가 정의된 객체**를 가리킴 (익명 클래스와 다름) |
| **예외 처리** | 람다 내부에서 발생한 예외는 호출하는 곳에서 처리 필요 |

---

## 📌 함께 알아두면 좋은 팁

### ① 메소드 레퍼런스

이미 정의된 메소드를 참조하면 람다를 더 간결하게 쓸 수 있다.

```java
Consumer<String> printer1 = s -> System.out.println(s);   // 람다식
Consumer<String> printer2 = System.out::println;          // 메소드 레퍼런스
```

### ② Optional과의 결합

값이 있을 수도 없을 수도 있는 상황을 람다와 함께 안전하게 다룬다.

```java
Optional<String> nullableValue = Optional.ofNullable(getNullableValue());

nullableValue.ifPresent(value -> System.out.println("Value: " + value));
String result = nullableValue.orElse("Default Value");
```

### ③ Stream API

컬렉션 처리를 **필터 → 변환 → 소비**의 파이프라인으로 표현한다.

```java
List<String> words = Arrays.asList("Apple", "Banana", "Orange", "Grapes");

words.stream()
     .filter(word -> word.startsWith("A"))   // A로 시작하는 것만
     .forEach(System.out::println);          // 출력
```

---

## 📝 정리

```
람다식(Lambda) — Java 8+
├─ 문법      (매개변수) -> { 본체 }   익명 클래스의 축약
├─ 대상      함수형 인터페이스(추상 메소드 1개)
├─ 활용      컬렉션 forEach, Comparator, Stream 등
├─ 주의      effectively final, this, 예외 처리
└─ 확장      메소드 레퍼런스(::) · Optional · Stream API
```

| 개념 | 한 줄 정의 |
|------|------|
| **람다식** | 함수형 인터페이스를 구현하는 익명 함수 |
| **함수형 인터페이스** | 추상 메소드가 하나뿐인 인터페이스 |
| **메소드 레퍼런스** | `Class::method`로 람다를 더 축약 |
| **Stream API** | 컬렉션을 파이프라인으로 처리 |

람다식은 "동작(코드)을 값처럼 전달"할 수 있게 해주는 도구다. 함수형 인터페이스·메소드 레퍼런스·Stream API와 결합하면, 자바 코드를 훨씬 간결하고 선언적으로 작성할 수 있다.
