---
title: "[Spring] @Bean과 static 주의점"
date: 2024-02-18 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, bean, static, singleton]
render_with_liquid: false
---

# **스프링 - `@Configuration`과 `@Bean`에서 static 주의 점**

스프링 프레임워크에서는 **`@Configuration`**과 **`@Bean`** 어노테이션을 사용하여 빈을 정의하고 관리합니다. 이 두 어노테이션은 스프링의 IoC (Inversion of Control) 컨테이너에게 빈의 생성 및 관리를 위임하는 역할을 합니다. 하지만 이러한 어노테이션을 사용할 때 static 키워드를 사용하는 것에 대한 주의점이 있습니다. 이번 글에서는 **`@Configuration`**과 **`@Bean`**에서 static 키워드를 사용할 때 주의해야 할 점에 대해 알아보겠습니다.

## ✅**static 키워드의 사용**

---

**`@Configuration`** 클래스 내부에서 **`@Bean`** 어노테이션을 사용하여 빈을 정의할 때 static 메서드로 빈을 정의하는 경우가 있습니다. 이렇게 정의된 static 메서드는 외부에서 직접 호출할 수 있으며, 해당 메서드가 반환하는 객체가 스프링 빈으로 등록됩니다. 이렇게 하면 스프링 IoC 컨테이너가 빈을 생성하고 관리하는 대신 개발자가 직접 빈을 생성할 수 있습니다.

예를 들어, 다음과 같이 **`@Configuration`** 클래스 내부에 static 메서드를 사용하여 빈을 정의할 수 있습니다.

```java
@Configuration
public class AppConfig {

    @Bean
    public static MyBean myBean() {
        return new MyBean();
    }
}

```

## ✅**static 메서드의 문제점**

---

static 메서드를 사용하여 빈을 정의하는 것은 몇 가지 문제점을 가지고 있습니다.

### **가. 단위 테스트 어려움**

static 메서드로 정의된 빈은 테스트하기 어렵습니다. 스프링 테스트 프레임워크에서는 static 메서드를 직접 호출하거나 모의 객체(Mock)를 만들어 static 메서드를 테스트하는 것이 어렵기 때문에 테스트가 복잡해질 수 있습니다.

### **나. 상태 공유 문제**

static 메서드로 정의된 빈은 싱글톤(Singleton) 스코프로 관리됩니다. 따라서 static 메서드가 가지고 있는 상태는 공유되어 모든 빈에 영향을 줄 수 있습니다. 이는 다중 스레드 환경에서 상태 공유 문제를 발생시킬 수 있습니다.

## ✅**해결 방법**

---

static 메서드를 사용하지 않고 인스턴스 메서드로 빈을 정의하면 이러한 문제를 해결할 수 있습니다. 인스턴스 메서드로 정의된 빈은 스프링 컨테이너가 빈의 인스턴스를 생성하고 관리하므로 테스트가 용이하고 상태 공유 문제도 해결됩니다.

따라서, **`@Configuration`** 클래스 내부에서 빈을 정의할 때는 static 키워드를 사용하지 않고 인스턴스 메서드를 사용하는 것이 좋습니다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}

```

## **4. 결론**

**`@Configuration`**과 **`@Bean`**을 사용하여 스프링 빈을 정의할 때는 static 키워드를 사용하는 것보다 인스턴스 메서드를 사용하는 것이 좋습니다. 이렇게 하면 테스트가 용이하고 상태 공유 문제를 방지할 수 있습니다. 개발자는 스프링의 강력한 기능을 최대한 활용하여 안정적이고 유지보수가 쉬운 애플리케이션을 개발할 수 있습니다.
