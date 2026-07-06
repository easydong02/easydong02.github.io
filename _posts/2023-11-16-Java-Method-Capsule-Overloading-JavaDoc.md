---
title: "[Java] 메서드 은닉화 다형성 자바Doc"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **메소드로 클래스 연결**, **은닉화(캡슐화)**, **오버로딩**, **생성자**, 그리고 문서 생성 도구 **JavaDoc**을 코드 중심으로 리뷰한다.

---

## 1. 은닉화(캡슐화) — 클래스를 메소드로 연결

세 클래스(`Hero`, `Bullet`, `UseHero`)를 메소드로 유기적으로 연결한다. 멤버변수를 `private`으로 감추고 **getter/setter로만 접근**하는 것이 은닉화다.

```java
// Hero 클래스
public class Hero {
    private int hp = 10;              // private으로 은닉
    private boolean fly = false;
    private String name = "메가맨";

    public void setHp(int hp)   { this.hp = hp; }   // setter
    public int  getHp()         { return hp; }      // getter
    public void setFly(boolean fly) { this.fly = fly; }
    public boolean getFly()     { return fly; }
    public void setName(String name) { this.name = name; }
    public String getName()     { return name; }

    public void fire(Bullet b) {                    // 다른 클래스를 매개변수로
        System.out.println("내가 사용하는 무기는 : " + b.getWeapon());
    }
}
```

```java
// Bullet 클래스 (다른 패키지)
public class Bullet {
    private String weapon = "칼";
    public void setWeapon(String weapon) { this.weapon = weapon; }
    public String getWeapon() { return weapon; }
}
```

```java
// UseHero 클래스
class UseHero {
    public static void main(String[] args) {
        Bullet b = new Bullet();
        Hero hero = new Hero();
        hero.setName("배트맨");
        hero.setHp(100);
        hero.setFly(true);
        System.out.println("히어로 이름: " + hero.getName());
        System.out.println("히어로 체력: " + hero.getHp());

        b.setWeapon("텀블러");
        hero.fire(b);   // Bullet의 weapon을 Hero의 fire로 전달
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/1.png)

> 💡 **은닉화(encapsulation)**: 데이터에 `private`을 두고 **메소드로만 값을 변경·호출**하는 방법.

> 💡 `hero.fire(b)` 대신 `hero.fire(new Bullet())`을 넣어도 될까? **된다.** 소괄호 안에는 `Bullet` 타입의 주소값만 있으면 되기 때문. 다만 `new Bullet()`은 기존 `b`와 다른 새 객체라 초기값 "칼"이 나온다.

---

## 2. 오버로딩(Overloading)

로직이 비슷한데 매번 새 메소드 이름을 짓는 건 오히려 개발에 장애가 된다. 그래서 자바는 **같은 이름의 메소드를 중복 정의**하게 해준다.

> **오버로딩 조건**: 메소드명은 **같되**, **매개변수의 자료형이나 개수가 달라야** 한다.

```java
public class Bird {
    public void fly()             { System.out.println("새가 날아간다."); }
    public void fly(int height)   { System.out.println("조금 더 높이 날아간다."); }
    public void fly(String stop)  { System.out.println("더욱 높이 날아간다."); }
}
```

매개변수에 무엇을 넣느냐에 따라 **어떤 메소드가 호출될지 결정**된다. 편리한 기능이다.

---

## 3. 생성자

클래스 이름과 같고 **반환 타입(void/int/String)이 없는** 메소드가 생성자다.

```java
public class Car {
    String color; int speed, price;

    public Car() {}                                   // 기본 생성자
    public Car(String color, int speed, int price) {  // 직접 생성자
        this.color = color; this.speed = speed; this.price = price;
    }

    public void main(String[] args) {
        Car c  = new Car("red", 400, 20000);   // 직접 생성자 사용
        Car c2 = new Car();                    // 기본 생성자 사용
    }
}
```

| 구분 | 설명 |
|------|------|
| **기본 생성자** | 선언 안 하면 컴파일러가 자동 생성. 단, 직접 생성자를 만들면 **자동 생성 안 됨** |
| **직접 생성자** | 객체 생성과 **동시에 속성 부여** (일일이 setter 호출 불필요) |

> 💡 `new Car(...)`의 `Car()`는 **선언이 아니라 사용**이다. 직접 생성자를 만들었으니 기본 생성자(`public Car(){}`)도 함께 만들어줘야 `new Car()`도 쓸 수 있다.

---

## 4. JavaDoc — 문서 자동 생성

> 실무에서 프로그램을 주고받을 때 **원본 `.java`를 주지 않는다**(기술 유출). 컴파일된 `.class`(바이너리)를 준다. 그런데 받는 사람은 그 안의 메소드·변수를 알 수 없다. 그래서 **설명서(README)**가 필요하고, 이를 자동 생성하는 도구가 **JavaDoc**이다.

```java
public class Smith {
    String name = "스미스";
    public int sal = 800;

    /**
     이 메서드는 우리 스미스의 급여를 변경하기 위함임
     **/
    public void setSal(int sal) { this.sal = sal; }
    public int getSal()         { return sal; }

    public void love(Dog d) {   // Dog도 자료형이므로 매개변수로 가능
        System.out.println("내 강아지 이름은 : " + d.getName());
    }
}
```

`javadoc` 명령으로 문서를 생성한다.

```bash
cd ...\member          # Smith.java가 있는 폴더
javadoc Smith.java     # HTML 문서 생성
```

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/3.png)

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/5.png)

`index.html`을 열면 **자바 공식 홈페이지 UI 그대로** 클래스의 멤버변수·메소드 정보가 나온다. 어노테이션 주석(`/** ~ */`)으로 원하는 설명도 넣을 수 있다.

---

## 📝 정리

```
메소드·은닉화·오버로딩·생성자·JavaDoc
├─ 은닉화     private + getter/setter로만 접근 (encapsulation)
├─ 오버로딩   같은 이름, 다른 매개변수 → 중복 정의
├─ 생성자     반환타입 없음, 객체 생성과 동시에 속성 부여
└─ JavaDoc    javadoc 명령으로 HTML 문서 자동 생성
```

| 개념 | 한 줄 정의 |
|------|------|
| **은닉화** | 데이터를 감추고 메소드로만 접근 |
| **오버로딩** | 매개변수가 다른 동명 메소드 |
| **생성자** | 객체 생성 시 초기화 담당 |
| **JavaDoc** | 소스에서 API 문서를 자동 생성 |

은닉화·오버로딩·생성자는 객체지향의 기둥이고, JavaDoc은 실무에서 협업의 필수 도구다. IDE 없이 원시적으로 다뤄보니 각 개념이 더 확실하게 손에 잡혔다.
