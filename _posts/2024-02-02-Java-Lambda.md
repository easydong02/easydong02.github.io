---
title: "[Java] 람다식(Lambda)"
date: 2024-02-02 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, lambda]
render_with_liquid: false
---

# **Java의 새로운 얼굴, 람다식(Lambda)**

Java 8에서 소개된 람다식은 기존의 자바 프로그래밍 패러다임을 크게 바꾸어 놓은 혁신적인 기능입니다. 람다식은 코드를 간결하게 만들어주고 함수형 프로그래밍의 특징을 도입함으로써 자바의 표현력을 향상시켰습니다. 이번 글에서는 람다식의 기본 개념, 활용 예시 및 주의사항에 대해 알아보겠습니다.

## ✅**람다식이란?**

---

람다식은 간단히 말해 익명 함수의 형태를 갖는 자바의 새로운 표현 방식입니다. 람다식은 주로 함수형 인터페이스를 구현할 때 사용되며, 코드를 간결하게 작성할 수 있도록 도와줍니다.

### **기본 문법**

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

위의 예시에서 볼 수 있듯이, 람다식은 익명 클래스를 훨씬 간결하게 표현할 수 있습니다.

### **매개변수와 반환값이 있는 람다식**

```java
// 기존의 익명 클래스
Comparator<Integer> comparator1 = new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
};

// 람다식
Comparator<Integer> comparator2 = (o1, o2) -> o1.compareTo(o2);

```

람다식을 사용하면 매개변수의 타입을 추론할 수 있기 때문에 타입을 명시적으로 적어주지 않아도 됩니다.

## ✅**람다식의 활용**

---

### **함수형 인터페이스와의 결합**

람다식은 주로 함수형 인터페이스를 구현할 때 활용됩니다. 함수형 인터페이스는 딱 하나의 추상 메서드만을 갖는 인터페이스로, 람다식과 잘 어울립니다.

```java
// 함수형 인터페이스
@FunctionalInterface
interface MyFunction {
    int calculate(int a, int b);
}

// 람다식을 활용한 함수형 인터페이스 구현
MyFunction add = (a, b) -> a + b;
MyFunction subtract = (a, b) -> a - b;

```

### **컬렉션 처리**

람다식은 컬렉션을 간결하게 처리하는 데에도 큰 도움을 줍니다.

```java
List<String> languages = Arrays.asList("Java", "Python", "JavaScript", "Kotlin");

// 기존의 방식
for (String language : languages) {
    System.out.println(language);
}

// 람다식을 사용한 방식
languages.forEach(language -> System.out.println(language));

```

람다식을 사용하면 반복문을 간결하게 표현할 수 있습니다.

## 📌**람다식의 주의사항**

---

- **함수형 인터페이스 여부 확인**: 람다식을 사용하려면 해당 람다식이 함수형 인터페이스를 구현하는지 확인해야 합니다.
- **변수의 final 또는 effectively final 여부**: 람다식에서 외부 변수를 사용할 경우 해당 변수는 final 또는 effectively final이어야 합니다.
- **this 키워드 주의**: 람다식 내부에서 **`this`** 키워드는 람다식이 정의된 객체를 가리킵니다. 주의가 필요한 경우에는 명시적으로 클래스 이름을 사용해야 합니다.
- **예외 처리**: 람다식 내부에서 예외가 발생할 경우, 람다식을 호출하는 곳에서 예외를 처리해주어야 합니다.

## 📌**추가로 알아두면 좋은 팁**

---

### **1. 메서드 레퍼런스 활용**

람다식을 더 간결하게 표현할 수 있는 방법 중 하나는 메서드 레퍼런스를 사용하는 것입니다. 메서드 레퍼런스를 사용하면 기존에 정의된 메서드를 간결하게 참조할 수 있습니다.

```java
// 람다식
Consumer<String> printer1 = s -> System.out.println(s);

// 메서드 레퍼런스
Consumer<String> printer2 = System.out::println;

```

### **2. Optional 활용**

**`Optional`**은 값이 있을 수도 없을 수도 있는 상황에서 사용하는 클래스로, 람다식과 함께 사용하면 코드의 안정성을 높일 수 있습니다.

```java
Optional<String> nullableValue = Optional.ofNullable(getNullableValue());

// 람다식을 사용한 값 존재 여부 확인
nullableValue.ifPresent(value -> System.out.println("Value is present: " + value));

// 값이 없을 경우 기본값 설정
String result = nullableValue.orElse("Default Value");

```

### **3. Stream API 활용**

Java의 Stream API는 컬렉션을 다루는 강력한 기능을 제공합니다. 람다식과 함께 사용하면 데이터 처리를 더 효율적으로 할 수 있습니다.

```java
List<String> words = Arrays.asList("Apple", "Banana", "Orange", "Grapes");

// 람다식과 Stream API를 사용한 필터링 및 출력
words.stream()
     .filter(word -> word.startsWith("A"))
     .forEach(System.out::println);

```
