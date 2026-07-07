---
title: "[DesignPattern] 팩토리 패턴"
date: 2023-12-22 00:00:00 +0900
categories: [Design-Pattern]
tags: [designpattern, factorypattern]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **팩토리 패턴(Factory Pattern)**을 다룬다. 무엇이든 될 수 있는 메타몽처럼, 객체 생성을 유연하게 다루는 패턴이다.

![Desktop View](/assets/img/Design-Pattern/Factory-Pattern/1.gif)

> **팩토리 패턴이란?**
> 객체를 사용하는 코드에서 **객체 생성 부분을 떼어내 추상화**한 패턴. 상위 클래스가 **뼈대**를 결정하고, 하위 클래스가 **구체적인 객체 생성**을 담당한다.

![Desktop View](/assets/img/Design-Pattern/Factory-Pattern/2.png)

**장점:**

| 장점 | 설명 |
|------|------|
| **느슨한 결합** | 상위·하위 클래스 분리, 생성 방식을 알 필요 없음 |
| **유연성** | 생성 로직이 분리되어 유연 |
| **유지보수성** | 생성 로직을 **한 곳만** 고치면 됨 |

---

## 1. 팩토리를 안 썼을 때 — 문제점

커피(추상) - 라떼/아메리카노(구현) 구조에서, 각 테스트마다 직접 `new`로 생성한다.

```java
// CoffeeTest
Coffee latte = new Latte(4000);
Coffee americano = new Americano(3000);

// CoffeeTest2
Coffee latte = new Latte(5000);   // 또 직접 new
```

> ⚠️ 만약 `Latte`의 **생성자가 바뀌면**, `new Latte(...)`를 쓰는 **모든 파일을 수정**해야 한다. 그래서 객체 생성을 책임질 **팩토리**가 필요하다.

---

## 2. 팩토리를 썼을 때

**객체 생성을 팩토리 클래스가 전담**한다.

```java
public class CoffeeFactory {
    public static Coffee getCoffee(String type) {
        if ("Latte".equalsIgnoreCase(type))         return new Latte(4000);
        else if ("Americano".equalsIgnoreCase(type)) return new Americano(4500);
        else                                          return new DefaultCoffee();  // 기본값
    }
}
```

```java
// 사용하는 쪽은 '무엇을 만들지'만 전달
Coffee latte = CoffeeFactory.getCoffee("Latte");        // 4000
Coffee americano = CoffeeFactory.getCoffee("Americano"); // 4500
Coffee unknown = CoffeeFactory.getCoffee("xyz");        // -1 (DefaultCoffee)
```

> 💡 이제 라떼·아메리카노 생성 로직이 바뀌어도 **팩토리 한 곳만** 고치면 된다. 사용하는 쪽은 **"어떤 것을 만들지"만 넘기면** 되니 결합도가 낮아진다.

> 💡 `DefaultCoffee`로 **잘못된 타입에도 안전하게** 기본값을 반환하는 것도 팩토리의 이점이다.

---

## 3. 한계와 확장

> ⚠️ 위 방식은 **가격이 다른 라떼**를 만들 수 없다. 객체 생성을 주관하는 팩토리가 하나뿐이기 때문. 이를 해결하려면 **팩토리 자체를 인터페이스/추상 클래스로** 만들어, 각각 개성 있는 팩토리(추상 팩토리 패턴)를 둔다.

---

## 📝 정리

```
팩토리 패턴
├─ 목적    객체 생성 로직을 분리·추상화
├─ 효과    느슨한 결합 + 유지보수성 (한 곳만 수정)
├─ 사용    Factory.getXxx(type)로 생성 위임
└─ 확장    팩토리를 추상화 → 추상 팩토리 패턴
```

| 개념 | 한 줄 정의 |
|------|------|
| **팩토리 패턴** | 객체 생성을 전담 클래스에 위임 |
| **느슨한 결합** | 생성 방식을 몰라도 됨 |
| **추상 팩토리** | 팩토리 자체를 추상화 |

팩토리 패턴의 핵심은 **"생성 로직을 한곳에 모아 변경에 강하게"**다. 객체 생성 방식이 바뀌어도 팩토리만 수정하면 되니, 유지보수성이 크게 향상된다.
