---
title: "[DesignPattern] SOLID원칙(SOLID)"
date: 2024-02-17 00:00:00 +0900
categories: [Design-Pattern]
tags: [designpattern, solid]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 객체지향 설계의 5대 원칙 **SOLID**를 예시와 함께 정리한다. 이 원칙들을 지키면 유연하고 확장 가능한 소프트웨어를 만들 수 있다.

| 약자 | 원칙 | 한 줄 요약 |
|:---:|------|------|
| **S** | 단일 책임 | 클래스는 책임 하나만 |
| **O** | 개방-폐쇄 | 확장엔 열고, 변경엔 닫기 |
| **L** | 리스코프 치환 | 자식은 부모를 대체 가능 |
| **I** | 인터페이스 분리 | 필요한 메소드만 의존 |
| **D** | 의존 역전 | 추상화에 의존 |

---

## 1. SRP — 단일 책임 원칙

> 한 클래스는 **변경되어야 하는 이유가 하나**여야 한다.

```java
// ❌ 여러 책임 (생성·이메일·리포트)
class UserManager {
    public void createUser() {}
    public void sendEmail() {}
    public void generateReport() {}
}

// ✅ 책임 분리
class UserManager     { public void createUser() {} }
class EmailService    { public void sendEmail() {} }
class ReportGenerator { public void generateReport() {} }
```

---

## 2. OCP — 개방-폐쇄 원칙

> **확장에는 열려 있고 변경에는 닫혀** 있어야 한다. 기존 코드 수정 없이 기능 추가.

```java
// ❌ 새 할인이 추가될 때마다 if 수정
class DiscountCalculator {
    double calc(double price, String type) {
        if (type.equals("Christmas")) return price * 0.2;
        else if (type.equals("NewYear")) return price * 0.1;
        return 0;
    }
}

// ✅ 인터페이스로 확장 (기존 코드 불변)
interface Discount { double calculateDiscount(double price); }
class ChristmasDiscount implements Discount { ... }
class NewYearDiscount implements Discount { ... }
class DiscountCalculator {
    double calc(double price, Discount discount) { return discount.calculateDiscount(price); }
}
```

---

## 3. LSP — 리스코프 치환 원칙

> 서브 타입은 언제나 **기반 타입으로 교체 가능**해야 한다.

```java
// ❌ Square extends Rectangle — setWidth가 height까지 바꿔 LSP 위반
// ✅ 공통 Shape를 상속
class Shape { protected int width, height; public int getArea() { return width * height; } }
class Rectangle extends Shape { ... }
class Square extends Shape { public void setSideLength(int l) { width = height = l; } }
```

> 💡 "정사각형은 직사각형이다"가 코드에서는 성립하지 않는다. 억지 상속은 LSP를 깨뜨린다.

---

## 4. ISP — 인터페이스 분리 원칙

> 클라이언트는 **사용하지 않는 메소드에 의존하지 말아야** 한다.

```java
// ❌ Robot이 eat()을 억지로 구현
interface Worker { void work(); void eat(); }

// ✅ 인터페이스 분리
interface Worker { void work(); }
interface Eater  { void eat(); }
class Programmer implements Worker, Eater { ... }
class Robot      implements Worker { }   // eat 불필요
```

---

## 5. DIP — 의존 역전 원칙

> 고수준·저수준 모듈 모두 **추상화에 의존**해야 한다. (세부 구현에 의존 X)

```java
// ❌ LightSwitch가 구체 클래스 LightBulb에 직접 의존
// ✅ 인터페이스(Switchable)에 의존
interface Switchable { void turnOn(); void turnOff(); }
class LightBulb implements Switchable { ... }
class LightSwitch {
    private Switchable device;
    public LightSwitch(Switchable device) { this.device = device; }  // 추상화 주입
}
```

> 💡 DIP는 **의존성 주입(DI)**의 이론적 기반이다. 스프링이 인터페이스에 구현체를 주입하는 것도 이 원칙을 따른다.

---

## 📝 정리

```
SOLID
├─ S 단일 책임   클래스는 이유 하나로만 변경
├─ O 개방-폐쇄   확장 O, 변경 X (인터페이스)
├─ L 리스코프    자식이 부모를 대체
├─ I 인터페이스  필요한 메소드만 분리
└─ D 의존 역전   추상화에 의존 (DI)
```

| 원칙 | 한 줄 정의 |
|------|------|
| **SRP** | 책임 하나 |
| **OCP** | 확장 열고 변경 닫기 |
| **LSP** | 자식=부모 대체 |
| **ISP** | 인터페이스 분리 |
| **DIP** | 추상화 의존 |

SOLID는 "변경에 강한 설계"를 위한 5가지 나침반이다. 공통 키워드는 **추상화(인터페이스)로 결합을 낮추는 것**이며, 이는 스프링의 DI·전략 패턴 등 실무 설계 전반의 토대가 된다.
