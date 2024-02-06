---
title: "[DesignPattern] 싱글턴 패턴(Singleton Pattern)"
date: 2024-02-06 00:00:00 +0900
categories: [Design-Pattern]
tags: [designpattern, singletonpattern]
render_with_liquid: false
---

# **싱글톤 패턴: 객체 지향 프로그래밍의 설계 패턴**

싱글톤 패턴은 객체 지향 프로그래밍에서 자주 사용되는 설계 패턴 중 하나입니다. 이 패턴은 특정 클래스의 인스턴스가 오직 하나만 생성되도록 보장하며, 그 인스턴스에 대한 전역적인 접근점을 제공합니다. 이 글에서는 싱글톤 패턴의 개념, 사용법, 예시 코드 등을 다뤄보겠습니다.

## ✅**싱글톤 패턴의 개념**

---

싱글톤 패턴은 다음과 같은 특징을 갖습니다:

1. **유일한 인스턴스**: 해당 클래스의 인스턴스가 오직 하나만 존재합니다.
2. **전역적인 접근성**: 어디서든지 해당 인스턴스에 접근할 수 있습니다.

싱글톤 패턴은 보통 다음과 같은 상황에서 사용됩니다:

- 공유된 자원에 대한 접근을 제어할 때
- 설정 관리자, 로그 레포터, 캐시 관리자 등의 서비스를 제공할 때

## ✅**싱글톤 패턴의 구현 방법**

---

싱글톤 패턴은 다양한 방식으로 구현할 수 있지만, 주로 다음 두 가지 방식으로 구현됩니다:

### **1. 정적 멤버 변수를 이용한 구현**

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {} // private constructor

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

### **2. 정적 팩토리 메서드를 이용한 구현**

```java
public class Singleton {
    private static final Singleton instance = new Singleton();

    private Singleton() {} // private constructor

    public static Singleton getInstance() {
        return instance;
    }
}

```

## ✅**싱글톤 패턴의 예시 코드**

---

### **Java 예시 코드**

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {} // private constructor

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    public void showMessage() {
        System.out.println("Hello, Singleton!");
    }
}

```

### **사용 예시**

```java
public class Main {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        singleton.showMessage();
    }
}

```

## 📌**싱글톤 패턴의 주의사항**

---

- 멀티스레드 환경에서 안전하게 사용하기 위해 동기화 처리를 해야 합니다.
- 리플렉션을 통한 객체 생성을 막아야 합니다.
- 직렬화와 역직렬화에 대한 보안 문제에 유의해야 합니다.