---
title: "[Java] 상속(Extends) 형변환(Casting)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **상속(extends)**과 **객체 간 형변환(casting)**을 다룬다. 문법 자체보다, 자바가 상속·생성자·오버라이딩을 통해 어떤 기능을 제공하는지에 초점을 둔다.

---

## 1. 상속 — 중복 제거

인류를 흑인·백인·황인 세 클래스로 나눈다고 하자. 멤버변수·메소드를 **일일이 다 만드는 건 프로그래밍의 본질(중복 제거)을 잃는 것**이다. 그래서 공통점을 담은 부모 클래스 `Man`을 만들고 상속시킨다.

```java
public class Man {
    String color;
    int age;

    public Man() {}
    public Man(int age) {
        System.out.println("Man생성자 호출");
        this.age = age;
    }
    public void walk() { System.out.println("걸을 수 있다."); }
    public void talk() { System.out.println("말할 수 있다."); }
}
```

공통 특징(직립보행·언어)은 부모에, 각 인종의 특징은 자식에 둔다.

```java
public class BlackMan extends Man {      // 흑인
    String color = "Black";
    int strength = 100;
    public void run() { System.out.println("흑인은 빨리 뛸 수 있다."); }
}

public class YellowMan extends Man {     // 황인
    String color = "Yellow";
    int height = 173;
}

public class WhiteMan extends Man {      // 백인
    public WhiteMan() {
        super(3);                        // 부모의 직접 생성자 호출
        String color = "white";
        String noseSize = "XL";
    }
}
```

> 💡 클래스 안에 올 수 있는 요소는 **변수(상태)**와 **메소드(기능)** 두 가지다. 각 인종의 고유 특징만 자식 클래스에 담고, 공통은 부모에서 상속받는다.

---

## 2. 상속과 생성자 (super)

```java
class UseMan {
    public static void main(String[] args) {
        WhiteMan wm = new WhiteMan();
        System.out.println(wm.color);
        wm.walk();    // Man의 메소드
        wm.talk();    // Man의 메소드
        System.out.println(wm.age);   // Man의 변수 (super(3)으로 3)
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Extends-Casting/1.png)

`wm`(백인)으로 `walk()`, `talk()`(부모 Man의 메소드)를 호출할 수 있다. 이유는 **상속과 생성자**에 있다.

> 💡 자식이 부모를 상속하면, 자식 생성자에 `super();`를 안 써도 **컴파일러가 자동으로 부모 생성자를 먼저 호출**한다. 그래서 부모가 먼저 생성되고("Man생성자 호출" 먼저 출력), 자식이 부모 필드에 접근할 수 있다. 부모의 직접 생성자를 쓰려면 `super(3)`처럼 매개변수를 맞춘다.

```
new WhiteMan()
   ↓
super(3) → Man(int age) 실행 → "Man생성자 호출", age=3
   ↓
WhiteMan 생성자 나머지 실행
```

---

## 3. 형변환과 다형성

부모 타입에 자식을 담는 **업 캐스팅**을 하면, 멤버변수와 메소드가 서로 다르게 동작한다.

```java
public class Bird {
    String name = "새";
    public void fly() { System.out.println("새가 날아요."); }
}
public class Duck extends Bird {
    String name = "오리";
    public void fly() { System.out.println("오리가 날아요."); }
}

class Use {
    public static void main(String[] args) {
        Bird b = new Duck();          // 업 캐스팅
        System.out.println(b.name);   // "새"        (멤버변수 → 부모)
        b.fly();                      // "오리가 날아요" (메소드 → 자식)
    }
}
```

> 💡 **자바(Sun)의 설계**: 업 캐스팅 시 **멤버변수는 부모**를, **메소드(기능)는 자식(오버라이딩된 것)**을 따른다. 그래서 `b.name`은 "새", `b.fly()`는 "오리가 날아요"가 나온다.

| 접근 | 따르는 것 | 결과 |
|------|:---:|------|
| `b.name` (변수) | **부모** (Bird) | "새" |
| `b.fly()` (메소드) | **자식** (Duck, 오버라이딩) | "오리가 날아요." |

부모 타입 객체인데 자식 기능이 나오는 것이 **오버라이딩**이고, 이것이 **다형성(polymorphism)**의 한 축이다.

---

## 📝 정리

```
상속 & 형변환
├─ 상속      공통은 부모, 특화는 자식 (중복 제거)
├─ super     자식 생성자는 부모 생성자를 먼저 자동 호출
├─ 업캐스팅  부모 타입에 자식 담기
└─ 다형성    변수는 부모, 메소드는 자식(오버라이딩)을 따름
```

| 개념 | 한 줄 정의 |
|------|------|
| **상속** | 공통 기능을 부모에 두고 물려받아 중복 제거 |
| **super()** | 부모 생성자 호출 (자동 삽입) |
| **업 캐스팅** | 자식을 부모 타입으로 담기 |
| **다형성** | 변수는 부모, 메소드는 자식을 따름 |

상속의 목적은 결국 **중복 제거**다. 그리고 업 캐스팅 시 "변수는 부모, 메소드는 자식"이라는 규칙이 다형성의 핵심 동작이다. 이 규칙을 잡으면 복잡한 상속 구조도 예측 가능해진다.
