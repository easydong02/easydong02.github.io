---
title: "[Java] 추상클래스 인터페이스"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

상속만으로는 "자식이 이 메소드를 **반드시** 구현하게 강제"할 수 없다. 이 강제성을 부여하는 것이 **추상 클래스**와 **인터페이스**다. 이번 글에서는 둘의 차이, 그리고 Adapter·마커 인터페이스·다중 상속·모호성까지 정리한다.

---

## 1. 추상 클래스 (abstract class)

> **추상 클래스란?**
> 구현되지 않은 메소드(**추상 메소드**)가 선언된 클래스. 자식이 **반드시 재정의(구현)**해야 메모리에 할당되므로, 구현을 **강제**하는 용도다.

```java
public abstract class Electronics {
    public abstract void on();       // 추상 메소드 (구현 X)
    public abstract void off();
    public final void printManual() { // 일반 메소드도 가능
        System.out.println("전자제품입니다.");
    }
}
```

추상 클래스를 상속한 자식은 **모든 추상 메소드를 오버라이딩**해야 한다.

```java
public class Refrigerator extends Electronics {
    @Override public void on()  { /* 냉장고만의 켜기 */ }
    @Override public void off() { /* 냉장고만의 끄기 */ }
}
```

![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/1.png)

> ⚠️ 구현하지 않으면 `The type Refrigerator must implement the inherited abstract method Electronics.on()` 오류가 난다.

> 💡 **왜 강제할까?** 같은 가전제품이라도 냉장고·청소기·세탁기는 전원을 켜고 끄는 방식이 다르다. 부모가 하나로 정해버리면 곤란하므로, **자식마다 자기 방식대로 구현하라**고 강제하는 것이다.

---

## 2. 인터페이스 (interface) — 순수한 '틀'

> **인터페이스란?**
> 추상 클래스를 고도화한 문법. **상수와 추상 메소드만** 존재한다(일반 메소드 불가). 클래스에 지정할 때는 `implements`를 쓰고, 구현은 지정받은 클래스에서 한다.

| 구분 | 추상 클래스 | 인터페이스 |
|------|:---:|:---:|
| 키워드 | `extends` | `implements` |
| 일반 메소드 | 가능 | 불가 (틀만) |
| 다중 지정 | 불가 | **가능** |

```java
public interface Animal {
    final static int eyes = 2;   // final·static 생략돼도 자동 적용 (공유)
    int legs = 4;

    void eat();
    void sleep();
    void getHand();
    void shakeTail();
}
```

```java
public class Cat implements Animal {
    @Override public void eat()       { System.out.println("참치를 먹는다."); }
    @Override public void sleep()     { System.out.println("잠을 잘 잔다."); }
    @Override public void getHand()   { System.out.println("손을 주지 않는다."); }
    @Override public void shakeTail() { System.out.println("꼬리를 흔들지 않는다."); }
}
```

---

## 3. Adapter — 필요한 것만 골라 구현

인터페이스를 직접 구현하면 **모든 메소드를 다 구현해야** 한다. 하지만 `Tiger`에서 `getHand`·`shakeTail`이 필요 없다면? **중간에 강제성을 없애주는 추상 클래스(Adapter)**를 둔다.

```
Animal(인터페이스) ──► AnimalAdapter(추상클래스, 모든 메소드 빈 구현)
                                    ▲ extends
                                  Tiger (필요한 것만 재정의)
```

```java
public abstract class AnimalAdapter implements Animal {
    @Override public void eat()       {}
    @Override public void sleep()     {}
    @Override public void getHand()   {}
    @Override public void shakeTail() {}
}

// Tiger는 인터페이스가 아니라 Adapter를 상속 → 원하는 것만 재정의
public class Tiger extends AnimalAdapter {
    @Override public void eat()   { /* ... */ }
    @Override public void sleep() { /* ... */ }
}
```

> 💡 Adapter는 관례상 클래스 이름 뒤에 **`Adapter`**를 붙여 목적을 드러낸다.

---

## 4. 마커 인터페이스 (Marker Interface)

내용이 **비어 있는** 인터페이스로, 클래스들을 **그룹화(타입 묶기)**하는 용도다.

> 넷플릭스의 작품은 "애니메이션이면서 영화", "스릴러이면서 드라마"처럼 여러 종류에 속한다. 하지만 자바는 다중 상속을 지원하지 않으므로, 마커 인터페이스로 여러 타입을 부여한다.

```java
public class Video {}
public interface Animation {}   // 마커 인터페이스 (비어 있음)

// Video를 상속하면서 Animation 타입도 획득
public class Onepiece extends Video implements Animation {}
```

`Onepiece`는 이제 `Video`이면서 `Animation`이기도 하다. 여러 타입으로 묶여 관리가 쉬워진다.

---

## 5. 다중 상속과 모호성

> 자바는 모호성 때문에 **클래스 다중 상속을 지원하지 않는다**(`extends A, B` 불가). 하지만 인터페이스는 `implements A, B`로 여러 개 지정할 수 있고, **JDK 8부터 `default` 메소드**로 인터페이스에도 메소드 본문을 작성할 수 있게 되어, 사실상 다중 상속을 허용한 셈이 됐다.

**모호성**: 여러 부모에 같은 이름의 메소드가 있으면 어느 것인지 알 수 없는 문제.

**상황 1 — 두 인터페이스에 같은 메소드** → 지정받은 클래스에서 재정의하고 `인터페이스.super`로 선택.

```java
public class ClassA implements InterA, InterB {
    @Override
    public void printData() {
        InterA.super.printData();   // 어느 인터페이스 것을 쓸지 명시
    }
}
```

**상황 2 — 부모 클래스 메소드 vs 인터페이스 default 메소드** → **클래스가 우선**.

```java
public class ClassC extends ClassB implements InterA {
    // ClassB(부모 클래스)의 printData()가 우선 실행됨
}
```

![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/2.png)
![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/3.png)

| 모호성 상황 | 해결 |
|------|------|
| 두 인터페이스에 같은 메소드 | 재정의 + `인터페이스.super.메소드()` |
| 부모 클래스 vs 인터페이스 default | **클래스가 우선** |

---

## 📝 정리

```
추상 클래스 & 인터페이스
├─ 추상 클래스  일반+추상 메소드, extends로 상속, 구현 강제
├─ 인터페이스   상수+추상 메소드만, implements, 다중 지정 가능
├─ Adapter      추상클래스로 강제성 완화 → 필요한 것만 재정의
├─ 마커         빈 인터페이스로 타입 그룹화
└─ 다중상속     인터페이스+default로 지원, 모호성은 규칙으로 해결
```

| 개념 | 한 줄 정의 |
|------|------|
| **추상 메소드** | 구현 없이 선언만 → 자식이 구현 강제 |
| **인터페이스** | 상수·추상 메소드만 있는 순수 틀 |
| **Adapter** | 강제성을 완화해 선택 구현을 돕는 추상 클래스 |
| **마커 인터페이스** | 타입 그룹화를 위한 빈 인터페이스 |

추상 클래스와 인터페이스 모두 "구현을 강제"하지만, 인터페이스는 다중 지정이 되고 순수한 틀이라는 점이 다르다. Adapter·마커 인터페이스는 그 강제성을 실무에서 유연하게 다루는 장치다.
