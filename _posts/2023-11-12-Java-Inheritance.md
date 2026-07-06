---
title: "[Java] 상속"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

핸드폰은 2G 폴더폰에서 3G, 4G, 지금의 5G로 발전했다. 세대가 올라갈 때마다 기존 기능은 그대로 두고 새 기능을 얹는다. 만약 새 세대가 나올 때마다 모든 기능을 처음부터 다시 만들라고 하면 개발자들이 퇴사할 것이다. 이 문제를 해결하는 개념이 **상속(inheritance)**이다.

> **상속이란?**
> 기존 클래스의 필드를 새 클래스가 **그대로 물려받는 것**. 공통 필드를 부모 클래스에 묶어두고, 자식 클래스가 이어받아 그 위에 특화된 기능을 얹는다.

---

## 1. 상속 문법

```java
class A {
    // A 필드
}

class B extends A {
    // A 필드 + B 필드
}
```

부모 클래스 `A`를 자식 클래스 `B`가 `extends A`로 상속하면, **B 안에 A의 필드가 고스란히** 들어간다. 부모·자식은 여러 이름으로 불린다.

| 클래스 | 다른 이름들 |
|:---:|------|
| **A (부모)** | 상위 클래스, 슈퍼 클래스, 기반 클래스 |
| **B (자식)** | 하위 클래스, 서브 클래스, 파생 클래스 |

---

## 2. super() — 부모 생성자 호출

> 자식 생성자를 호출하면 자식 필드만 메모리에 올라갈 것 같지만, 사실 **자식 생성자는 항상 부모 생성자를 먼저 호출**한다. 그래서 부모·자식 필드가 **모두** 메모리에 할당된다. 이때 부모 생성자를 부르는 것이 **`super()`**다.

```java
class A {
    public A() {
        System.out.println("부모 생성자 호출");
    }
    int aData = 100;
    void printAData() { System.out.println(aData); }
    void printData()  { System.out.println(aData); }
}

class B extends A {
    public B() {
        super();   // 생략 가능 (생략 시 컴파일러가 자동으로 넣어줌)
        System.out.println("자식 생성자 호출");
    }
    int bData = 10000;
    int getAData() { return aData; }   // 부모 필드 사용 가능
    int getBData() { return bData; }
}

public class InheritanceTest {
    public static void main(String[] args) {
        B b = new B();                    // 자식 생성자만 호출
        b.printData();                    // 부모 메소드 사용
        System.out.println(b.getAData()); // 자식 메소드가 부모 변수 반환
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Inheritance/1.png)

```
new B()  ──►  super() 자동 호출  ──►  "부모 생성자 호출"
                                       "자식 생성자 호출"
```

자식 생성자만 썼는데 **부모 생성자까지 호출**됐다. 부모 필드를 쓰려면 부모 생성자가 실행되어야 하기 때문이다. 그 결과 자식 객체로 부모 메소드도, 부모 변수도 사용할 수 있다.

---

## 3. 오버라이딩 (재정의)

부모의 메소드가 자식에게 안 어울릴 때, **자식에서 같은 이름으로 다시 정의**하는 것이 오버라이딩이다.

> **동작 원리**: 부모 필드가 먼저 메모리에 올라간다. 자식이 같은 이름의 메소드를 재정의하면, **새로 만드는 게 아니라 기존 메소드 저장소의 주소를 자식 코드로 덮어쓴다.** 그래서 자식 객체로 접근하면 재정의된 내용이 읽힌다.

사실 이건 우리가 아는 대입과 같은 개념이다.

```java
int a = 100;
a = 200;   // 이후 a를 출력하면? → 200 (나중 값이 덮어씀)
```

오버라이딩도 마찬가지로, 부모·자식에 같은 이름의 필드가 있으면 **나중의 자식 필드가 사용**된다.

```java
class Car {
    String brand, color;
    int price;
    public Car() {}
    public Car(String brand, String color, int price) {
        super();
        this.brand = brand; this.color = color; this.price = price;
    }
    void engineStart() { System.out.println("열쇠로 시동 켜기"); }
    void engineStop()  { System.out.println("열쇠로 시동 끄기"); }
}

class SuperCar extends Car {
    String mode;

    @Override
    void engineStart() {                        // 부모 메소드 재정의
        System.out.println("음성으로 시동 킴");
    }
    void openRoof()  { System.out.println("썬루프 열림"); }
    void closeRoof() { System.out.println("썬루프 닫힘"); }
}

public class Road {
    public static void main(String[] args) {
        SuperCar ferrari = new SuperCar();
        ferrari.engineStart();   // "음성으로 시동 킴" (재정의된 것)
        ferrari.openRoof();
    }
}
```

슈퍼카가 열쇠로 시동을 켜는 건 어울리지 않으니, `engineStart()`를 "음성으로 시동 킴"으로 재정의했다.

> 💡 **`@Override`**는 "이 메소드는 부모 것을 재정의한다"는 표시다. 상속받되 필요한 메소드만 **선택적으로** 바꿀 수 있다는 게 오버라이딩의 매력이다.

---

## 📝 정리

```
상속(inheritance)
├─ 문법     class 자식 extends 부모  → 부모 필드 물려받음
├─ super()  자식 생성자는 부모 생성자를 먼저 호출 (생략 가능)
├─ 재사용   자식 객체로 부모 필드·메소드 사용 가능
└─ 오버라이딩  @Override로 부모 메소드를 선택적으로 재정의
```

| 개념 | 한 줄 정의 |
|------|------|
| **상속** | 부모의 필드를 자식이 물려받는 것 |
| **super()** | 부모 생성자 호출 (자동 삽입됨) |
| **오버라이딩** | 부모 메소드를 자식에서 재정의 |

상속은 "공통은 부모에, 특화는 자식에"라는 재사용의 핵심 도구다. `super()`로 부모가 함께 초기화된다는 점, 그리고 `@Override`로 필요한 부분만 바꿀 수 있다는 점을 기억하자.
