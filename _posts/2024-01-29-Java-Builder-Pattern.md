---
title: "[Java] 빌더 패턴(Builder Pattern)"
date: 2024-01-29 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, designpattern, builderpattern]
render_with_liquid: false
---

# **Java - Builder 패턴: 객체 생성을 더 편리하게**

## ✅**소개**

---

Java에서 객체를 생성할 때 많은 매개변수를 가진 생성자를 사용하는 경우가 있습니다. 이때, 가독성이 떨어지고 매개변수의 순서를 실수할 수 있는 등의 문제가 발생할 수 있습니다. 이러한 문제를 해결하고자 Builder 패턴이 등장했습니다. Builder 패턴은 객체를 생성하는 복잡한 과정을 추상화하고, 가독성과 확장성을 향상시킵니다. 이번 글에서는 Java에서 Builder 패턴을 어떻게 활용하는지에 대해 알아보겠습니다.

## ✅**Builder 패턴이란?**

---

Builder 패턴은 객체의 생성 과정을 나타내는 클래스를 정의하고, 이를 통해 다양한 속성을 가진 객체를 생성할 수 있도록 하는 패턴입니다. 일반적으로 다양한 매개변수를 가진 생성자를 사용하는 것보다 가독성이 좋고, 필요한 속성만 설정할 수 있어 편리합니다.

## ✅**Builder 패턴의 장점**

---

1. **가독성 향상:** 복잡한 객체 생성 로직을 숨기고, 각 속성을 메서드로 설정할 수 있어 가독성이 향상됩니다.
2. **확장성:** 새로운 속성이 추가될 때 기존 코드를 변경하지 않고 쉽게 추가할 수 있습니다.
3. **불변성 유지:** 객체를 불변하게 만들 수 있어, 다중 스레드 환경에서 안전하게 사용할 수 있습니다.

## ✅**Builder 패턴 예시**

---

### **1. 간단한 예시**

```java
public class Car {
    private String brand;
    private String model;
    private int year;
    private String color;

    private Car(Builder builder) {
        this.brand = builder.brand;
        this.model = builder.model;
        this.year = builder.year;
        this.color = builder.color;
    }

    public static class Builder {
        private String brand;
        private String model;
        private int year;
        private String color;

        public Builder(String brand, String model) {
            this.brand = brand;
            this.model = model;
        }

        public Builder year(int year) {
            this.year = year;
            return this;
        }

        public Builder color(String color) {
            this.color = color;
            return this;
        }

        public Car build() {
            return new Car(this);
        }
    }
}

// 사용 예시
Car car = new Car.Builder("Toyota", "Camry")
                .year(2022)
                .color("Blue")
                .build();

```

위의 예시에서 **`Car`** 클래스는 빌더 패턴을 적용한 예시입니다. **`Car`** 클래스의 인스턴스를 생성할 때 **`Builder`** 클래스를 사용하여 각 속성을 설정할 수 있습니다.

### **2. Lombok을 활용한 더 간결한 예시**

Lombok 라이브러리를 사용하면 더 간결하게 Builder 패턴을 구현할 수 있습니다.mport lombok.Builder;
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

```java

```

Lombok의 **`@Builder`** 애노테이션을 사용하면 Builder 클래스를 자동으로 생성해주어 코드의 중복을 피할 수 있습니다.

## ✅**더 나아가기: Fluent Interface와 더불어**

---

Builder 패턴을 적용하면 객체 생성 코드가 명확하고 가독성이 좋아집니다. 그러나 더 나아가서 Fluent Interface를 활용하면 코드를 더 자연스럽게 작성할 수 있습니다. Fluent Interface는 메서드 체이닝을 통해 메서드 호출을 연결하여 자연스러운 문장처럼 보이게 하는 스타일입니다.

아래는 Fluent Interface를 Builder 패턴에 적용한 예시입니다.

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

        public CarBuilder brand(String brand) {
            car.brand = brand;
            return this;
        }

        public CarBuilder model(String model) {
            car.model = model;
            return this;
        }

        public CarBuilder year(int year) {
            car.year = year;
            return this;
        }

        public CarBuilder color(String color) {
            car.color = color;
            return this;
        }

        public Car build() {
            return car;
        }
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

Fluent Interface를 활용하면 메서드 호출이 연결되어 자연스러운 문장 형태로 코드를 작성할 수 있습니다. 이는 가독성을 향상시키고 코드 작성을 더 즐겁게 만들어줍니다.
