---
title: "[Java] 캐스팅 변수"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **업 캐스팅(up casting)·다운 캐스팅(down casting)**과 **변수의 종류**(지역·전역·정적)를 다룬다. 시작 전에 대전제 하나를 기억하자.

> **"모든 자식은 부모 타입이다."**
> 상속에서 배웠듯, 모든 자식 클래스는 부모 타입이기도 하다. 그래서 자식을 부모 타입에 담는 **업 캐스팅이 가능**하다.

---

## 1. 업 캐스팅 · 다운 캐스팅

| 종류 | 정의 |
|------|------|
| **업 캐스팅** | 자식 값을 **부모 타입**으로 형변환 |
| **다운 캐스팅** | 업 캐스팅된 객체를 다시 **자식 타입**으로 형변환(복원) |

> ⚠️ **부모 값은 자식 타입에 담을 수 없다.** 다운 캐스팅은 "업 캐스팅했던 것을 되돌리는 것"이지, 순수 부모 객체를 자식으로 바꾸는 게 아니다.

**왜 굳이 부모 타입으로 바꿀까?** 여러 자식을 한 저장 공간으로 받으려면 **동일한 타입**이어야 한다. 자식끼리는 타입이 달라 한 번에 못 받지만, 모두 부모 타입이므로 **부모 타입 하나로 여러 자식을 담을 수 있다.** 자식 고유 기능이 필요하면 다운 캐스팅으로 복원한다. 즉 **여러 자식 클래스를 효율적으로 관리**하기 위함이다.

```java
class Car {
    String brand, color; int price;
    public Car() {}
    public Car(String brand, String color, int price) {
        super(); this.brand = brand; this.color = color; this.price = price;
    }
    void engineStart() { System.out.println("열쇠로 시동 킴"); }
    void engineStop()  { System.out.println("열쇠로 시동 끔"); }
}

class SuperCar extends Car {
    String mode;
    public SuperCar() {}
    public SuperCar(String brand, String color, int price, String mode) {
        super(brand, color, price); this.mode = mode;
    }
    @Override
    void engineStart() { super.engineStart(); System.out.println("음성으로 시동 킴"); }
    void openRoof()  { System.out.println("뚜껑 열림"); }
    void closeRoof() { System.out.println("뚜껑 닫힘"); }
}
```

```java
public class CastingTest {
    public static void main(String[] args) {
        Car matiz = new Car();
        SuperCar ferrari = new SuperCar();

        // 업 캐스팅: 부모 타입에 자식 생성자
        Car noOptionFerrari = new SuperCar();
        noOptionFerrari.engineStart();

        // SuperCar brokenFerrari = (SuperCar)new Car();  // ❌ 오류(순수 부모→자식 불가)

        // 다운 캐스팅: 복원
        SuperCar fullOptionFerrari = (SuperCar) noOptionFerrari;
        fullOptionFerrari.openRoof();   // 자식 고유 메소드 사용 가능
    }
}
```

> 💡 생성자는 메소드가 아니라 **값**(필드를 메모리에 할당하고 주소값을 갖는)이다. 그래서 `Car noOptionFerrari = new SuperCar();`처럼 부모 타입 변수에 자식 값을 대입할 수 있다. 단, 이 상태에선 `openRoof()`(자식 고유)가 안 보인다. 다운 캐스팅으로 복원해야 쓸 수 있다.

### instanceof — 객체 타입 비교

`a instanceof B`는 `a`가 `B` 타입이면 true, 아니면 false다.

| 식 | 결과 | 이유 |
|------|:---:|------|
| `matiz instanceof Car` | true | 자기 타입 |
| `matiz instanceof SuperCar` | **false** | 부모는 자식 타입 아님 |
| `ferrari instanceof Car` | true | **자식은 부모 타입** |
| `ferrari instanceof SuperCar` | true | 자기 타입 |
| `noOptionFerrari instanceof SuperCar` | true | 업 캐스팅돼도 본질은 자식 |

---

## 2. 변수의 종류

| 종류 | 정의 |
|------|------|
| **지역변수** | 지역 안에서 선언된 변수 (클래스 지역 제외) |
| **매개변수** | 메소드 선언 시 소괄호 안에 선언 |
| **전역변수** | 전 지역에서 사용 (클래스의 인스턴스 변수) |
| **정적변수(static)** | **객체 간 공유**, 편의성 |

### 전역변수 vs 정적변수 — 핵심 차이

같은 패키지에 두 클래스를 두고 실험해보자.

```java
public class Variable1 {
    int data;          // 전역(인스턴스) 변수
    static int data_s; // 정적 변수
}
```

**전역변수** — `new`로 생성자를 재정의하면 새 주소가 할당돼 **0부터 다시 시작**한다.

```java
Variable1 v1 = new Variable1();
v1.data++; ... v1.data++;   // 5번
v1 = new Variable1();       // 새 객체(새 주소)
v1.data++; ... v1.data++;   // 다시 5번
System.out.println(v1.data);  // → 5
```

**정적변수** — 생성자가 아니라 **컴파일러가 메모리에 올려두므로**, 생성자를 몇 번 재정의해도 값이 유지되고 **다른 객체로 접근해도 공유**된다.

```java
System.out.println(v1.data_s);  // → 10 (누적 유지)
System.out.println(v2.data_s);  // → 10 (v2로 접근해도 같음)
```

> 💡 정적변수는 객체와 무관하므로, 표현할 때 객체명이 아니라 **클래스명**을 쓴다. `Variable1.data_s++;`

---

## 3. 저장 기억 부류 & 접근 제어자

| 구분 | 지역변수·매개변수 | 전역변수 | 정적변수 |
|------|:---:|:---:|:---:|
| **메모리 영역** | Stack | Data | Data |
| **초기화** | 직접 | 자동 | 자동 |
| **생명주기** | 블록(`}`) 종료 시 | `new` 시 초기화 | 프로그램 종료 시 |

**접근 권한 제어자**는 필드의 접근 범위를 정한다.

| 제어자 | 접근 범위 |
|------|------|
| `default` | 같은 패키지만 (그냥 선언 시 기본값) |
| `public` | **모든 곳** |
| `protected` | 같은 패키지 + **자식은 다른 패키지에서도** |
| `private` | 같은 클래스만 (간접 접근용) |

---

## 📝 정리

```
캐스팅 & 변수
├─ 업 캐스팅   자식 → 부모 타입 (여러 자식을 하나로 관리)
├─ 다운 캐스팅 부모 타입 객체 → 자식 타입 복원
├─ instanceof  타입 비교 (자식은 부모 타입에도 true)
├─ 정적변수    static, 객체 간 공유 (클래스명으로 접근)
└─ 접근 제어자 public > protected > default > private
```

| 개념 | 한 줄 정의 |
|------|------|
| **업/다운 캐스팅** | 자식↔부모 타입 형변환 |
| **instanceof** | 객체가 특정 타입인지 검사 |
| **static** | 모든 객체가 공유하는 변수 |
| **접근 제어자** | 필드의 접근 범위를 지정 |

"모든 자식은 부모 타입"이라는 대전제가 업/다운 캐스팅의 핵심이다. 그리고 `static`은 객체가 아니라 **클래스에 소속**되어 공유된다는 점을 기억하면, 다형성과 변수 관리가 한결 명확해진다.
