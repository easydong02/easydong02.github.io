---
title: "[DesignPattern] SOLID원칙(SOLID)"
date: 2024-02-17 00:00:00 +0900
categories: [Design-Pattern]
tags: [designpattern, solid]
render_with_liquid: false
---

# **SOLID 원칙: 객체지향 설계의 기본 원칙**

SOLID는 객체지향 설계 원칙의 앞글자를 딴 것으로, 이를 통해 유연하고 확장 가능한 소프트웨어를 개발할 수 있습니다. 이번 글에서는 SOLID 원칙에 대해 자세히 알아보고, 각각의 원칙을 예시 코드와 함께 살펴보겠습니다.

## ✅**SRP: Single Responsibility Principle (단일 책임 원칙)**

---

한 클래스는 단 하나의 책임을 가져야 한다는 원칙입니다. 이는 클래스가 변경되어야 하는 이유가 하나여야 함을 의미합니다. 만약 클래스가 여러 책임을 갖게 되면, 한 책임이 변경되면 다른 책임에도 영향을 미칠 수 있습니다.

예시 코드를 통해 살펴보겠습니다.

```java
// 잘못된 예시
class UserManager {
    public void createUser() {
        // 사용자 생성 로직
    }

    public void sendEmail() {
        // 이메일 전송 로직
    }

    public void generateReport() {
        // 리포트 생성 로직
    }
}

```

위 코드는 단일 책임 원칙을 위반합니다. createUser, sendEmail, generateReport 메서드는 각각 사용자 생성, 이메일 전송, 리포트 생성과 관련된 다른 책임을 갖고 있습니다. 이를 개선하기 위해 각 책임에 해당하는 클래스를 만들어야 합니다.

```java
// 개선된 예시
class UserManager {
    public void createUser() {
        // 사용자 생성 로직
    }
}

class EmailService {
    public void sendEmail() {
        // 이메일 전송 로직
    }
}

class ReportGenerator {
    public void generateReport() {
        // 리포트 생성 로직
    }
}

```

위 코드에서는 각 클래스가 하나의 책임을 갖도록 분리하여 변경 사항이 다른 클래스에 영향을 미치지 않도록 했습니다.

## ✅**OCP: Open/Closed Principle (개방-폐쇄 원칙)**

---

소프트웨어 요소는 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다는 원칙입니다. 즉, 기존의 코드를 수정하지 않고 새로운 기능을 추가할 수 있어야 합니다.

예시 코드를 통해 살펴보겠습니다.

```java
// 잘못된 예시
class DiscountCalculator {
    public double calculateDiscount(double price, String discountType) {
        if (discountType.equals("Christmas")) {
            return price * 0.2;
        } else if (discountType.equals("NewYear")) {
            return price * 0.1;
        }
        return 0;
    }
}

```

위 코드는 새로운 할인 유형이 추가될 때마다 DiscountCalculator 클래스를 변경해야 합니다. 이를 개선하기 위해 확장에는 열려있고 변경에는 닫혀있도록 설계해야 합니다.

```java
// 개선된 예시
interface Discount {
    double calculateDiscount(double price);
}

class ChristmasDiscount implements Discount {
    @Override
    public double calculateDiscount(double price) {
        return price * 0.2;
    }
}

class NewYearDiscount implements Discount {
    @Override
    public double calculateDiscount(double price) {
        return price * 0.1;
    }
}

class DiscountCalculator {
    public double calculateDiscount(double price, Discount discount) {
        return discount.calculateDiscount(price);
    }
}

```

위 코드에서는 Discount 인터페이스를 도입하여 새로운 할인 유형을 추가할 때 Discount 인터페이스를 구현하는 새로운 클래스를 만들기만 하면 되므로 기존 코드를 변경할 필요가 없습니다.

## ✅**LSP: Liskov Substitution Principle (리스코프 치환 원칙)**

---

서브 타입은 언제나 자신의 기반 타입으로 교체할 수 있어야 한다는 원칙입니다. 즉, 서브 클래스는 기반 클래스의 기능을 사용할 수 있어야 하며, 클라이언트 코드는 이를 알아차리지 못해야 합니다.

예시 코드를 통해 살펴보겠습니다.

```java
// 잘못된 예시
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}

```

위 코드에서는 Square 클래스가 Rectangle 클래스의 서브 클래스이지만, Square는 사실상 직사각형이 아닌 정사각형을 나타내기 때문에 LSP를 위반합니다.

```java
// 개선된 예시
class Shape {
    protected int width;
    protected int height;

    public int getArea() {
        return width * height;
    }
}

class Rectangle extends Shape {
    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }
}

class Square extends Shape {
    public void setSideLength(int length) {
        this.width = length;
        this.height = length;
    }
}

```

위 코드에서는 직사각형(Rectangle)과 정사각형(Square)이 모두 Shape 클래스를 상속받아 getArea() 메서드를 사용할 수 있도록 하였습니다.

## ✅**ISP: Interface Segregation Principle (인터페이스 분리 원칙)**

---

클라이언트가 자신이 사용하지 않는 메서드에 의존하지 않아야 한다는 원칙입니다. 즉, 인터페이스는 그 인터페이스를 구현하는 클라이언트에게 필요한 메서드만 제공해야 합니다.

예시 코드를 통해 살펴보겠습니다.

```java
// 잘못된 예시
interface Worker {
    void work();
    void eat();
}

class Programmer implements Worker {
    @Override
    public void work() {
        // 프로그래머의 작업
    }

    @Override
    public void eat() {
        // 프로그래머의 식사
    }
}

class Robot implements Worker {
    @Override
    public void work() {
        // 로봇의 작업
    }

    @Override
    public void eat() {
        // 로봇은 식사를 할 필요가 없음
    }
}

```

위 코드에서는 Robot 클래스가 eat() 메서드를 구현하지만, 로봇은 식사를 할 필요가 없기 때문에 이는 ISP를 위반합니다.

```java
// 개선된 예시
interface Worker {
    void work();
}

interface Eater {
    void eat();
}

class Programmer implements Worker, Eater {
    @Override
    public void work() {
        // 프로그래머의 작업
    }

    @Override
    public void eat() {
        // 프로그래머의 식사
    }
}

class Robot implements Worker {
    @Override
    public void work() {
        // 로봇의 작업
    }
}

```

위 코드에서는 Worker 인터페이스와 Eater 인터페이스를 분리하여 각각의 인터페이스에 필요한 메서드만 구현하도록 하였습니다.

## ✅**DIP: Dependency Inversion Principle (의존 역전 원칙)**

---

의존 역전 원칙은 고수준 모듈은 저수준 모듈에 의존해서는 안 되며, 둘 모두 추상화에 의존해야 한다는 원칙입니다. 즉, 추상화에 의존하고 세부 구현에는 의존하지 않아야 합니다. 이는 고수준 모듈이 변경되더라도 저수준 모듈의 변경 없이도 올바르게 작동하도록 하는 것을 의미합니다.

예시 코드를 통해 살펴보겠습니다.

```java
// 잘못된 예시
class LightBulb {
    public void turnOn() {
        // 전구를 켜는 로직
    }

    public void turnOff() {
        // 전구를 끄는 로직
    }
}

class LightSwitch {
    private LightBulb lightBulb;

    public LightSwitch() {
        this.lightBulb = new LightBulb();
    }

    public void toggle() {
        if (lightBulb.isOn()) {
            lightBulb.turnOff();
        } else {
            lightBulb.turnOn();
        }
    }
}

```

위 코드에서는 LightSwitch 클래스가 LightBulb 클래스에 직접 의존하고 있으므로 DIP를 위반합니다.

```java
// 개선된 예시
interface Switchable {
    void turnOn();
    void turnOff();
}

class LightBulb implements Switchable {
    @Override
    public void turnOn() {
        // 전구를 켜는 로직
    }

    @Override
    public void turnOff() {
        // 전구를 끄는 로직
    }
}

class LightSwitch {
    private Switchable device;

    public LightSwitch(Switchable device) {
        this.device = device;
    }

    public void toggle() {
        if (device.isOn()) {
            device.turnOff();
        } else {
            device.turnOn();
        }
    }
}

```

위 코드에서는 LightSwitch 클래스가 Switchable 인터페이스에 의존하도록 개선되었습니다. 이렇게 하면 LightSwitch 클래스는 구체적인 LightBulb 클래스에 직접 의존하지 않고, Switchable 인터페이스에 의존하게 되므로 의존 역전 원칙을 준수하게 됩니다.
