---
title: "[Java] 빌더 패턴(Builder Pattern)"
date: 2024-01-29 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, designpattern, builderpattern]
render_with_liquid: false
---

## 📌 들어가며

매개변수가 많은 생성자로 객체를 만들다 보면 `new Car("Toyota", "Camry", 2022, "Blue", ...)`처럼 되어, **어떤 값이 무슨 인자인지** 헷갈리고 순서를 실수하기 쉽다. 이 문제를 해결하는 것이 **Builder 패턴**이다.

> **Builder 패턴이란?**
> 객체의 생성 과정을 별도 클래스로 분리해, 다양한 속성을 가진 객체를 **단계적으로** 생성하는 패턴. 필요한 속성만 골라 설정할 수 있어 가독성이 좋다.

**장점 3가지:**

| 장점 | 설명 |
|------|------|
| **가독성 향상** | 각 속성을 메소드로 설정 → 무슨 값을 넣는지 명확 |
| **확장성** | 새 속성 추가 시 기존 코드 변경 최소화 |
| **불변성 유지** | 객체를 불변하게 만들어 멀티스레드에서 안전 |

---

## 1. 기본 예시

정적 내부 클래스 `Builder`를 두고, 각 설정 메소드가 **자기 자신(`this`)을 반환**해 체이닝을 가능하게 한다.

```java
public class Car {
    private String brand;
    private String model;
    private int year;
    private String color;

    private Car(Builder builder) {           // 생성자는 private
        this.brand = builder.brand;
        this.model = builder.model;
        this.year  = builder.year;
        this.color = builder.color;
    }

    public static class Builder {
        private String brand;
        private String model;
        private int year;
        private String color;

        public Builder(String brand, String model) {   // 필수값은 생성자로
            this.brand = brand;
            this.model = model;
        }

        public Builder year(int year)     { this.year = year;   return this; }  // 선택값
        public Builder color(String color){ this.color = color; return this; }

        public Car build() {                 // 최종 객체 생성
            return new Car(this);
        }
    }
}

// 사용 예시
Car car = new Car.Builder("Toyota", "Camry")   // 필수
                .year(2022)                     // 선택
                .color("Blue")                  // 선택
                .build();
```

> 💡 **필수값은 Builder 생성자**로, **선택값은 체이닝 메소드**로 받는 것이 일반적인 패턴이다.

---

## 2. Lombok으로 더 간결하게

Lombok의 **`@Builder`** 애노테이션을 쓰면 Builder 클래스를 자동 생성해준다.

```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class Car {
    private String brand;
    private String model;
    private int year;
    private String color;
}

// 사용 예시
Car car = Car.builder()
             .brand("Toyota")
             .model("Camry")
             .year(2022)
             .color("Blue")
             .build();
```

애노테이션 하나로 위의 장황한 Builder 코드를 대체하니, **중복을 크게 줄일 수 있다.**

---

## 3. 한 걸음 더 — Fluent Interface

Builder에 **Fluent Interface**(메소드 체이닝으로 문장처럼 읽히는 스타일)를 적용하면 코드가 더 자연스러워진다.

```java
public class Car {
    private String brand;
    private String model;
    private int year;
    private String color;

    private Car() {}

    public static CarBuilder builder() {
        return new CarBuilder();
    }

    public static class CarBuilder {
        private final Car car = new Car();
        private CarBuilder() {}

        public CarBuilder brand(String brand) { car.brand = brand; return this; }
        public CarBuilder model(String model) { car.model = model; return this; }
        public CarBuilder year(int year)      { car.year = year;   return this; }
        public CarBuilder color(String color) { car.color = color; return this; }

        public Car build() { return car; }
    }
}

// 사용 예시
Car car = Car.builder()
             .brand("Toyota")
             .model("Camry")
             .year(2022)
             .color("Blue")
             .build();
```

메소드 호출이 연결되어 **자연스러운 문장 형태**로 읽히니, 가독성이 좋아지고 코드 작성도 즐거워진다.

---

## 📝 정리

```
Builder 패턴
├─ 목적    매개변수 많은 생성자의 가독성·실수 문제 해결
├─ 구조    내부 Builder 클래스 + 체이닝(return this) + build()
├─ 필수/선택  필수값은 생성자, 선택값은 체이닝 메소드
├─ Lombok  @Builder로 자동 생성 (중복 제거)
└─ 확장    Fluent Interface로 문장처럼 읽히게
```

| 개념 | 한 줄 정의 |
|------|------|
| **Builder 패턴** | 객체 생성을 단계별로 분리하는 디자인 패턴 |
| **체이닝(return this)** | 설정 메소드가 자신을 반환해 연결 |
| **@Builder** | Lombok이 Builder를 자동 생성 |

Builder 패턴은 "생성자 지옥"을 벗어나게 해주는 실용적인 패턴이다. 순수 자바로 구현할 수도 있지만, 실무에서는 Lombok의 `@Builder` 한 줄로 간결하게 쓰는 경우가 많다.
