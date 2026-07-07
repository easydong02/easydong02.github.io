---
title: "[Spring] 스프링 컨테이너(Spring Container)"
date: 2024-02-18 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, container, bean, applicationcontext]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 스프링의 핵심인 **스프링 컨테이너**의 개념·종류·동작 방식을 정리한다.

> **스프링 컨테이너란?**
> 객체의 **라이프사이클을 관리**하고 **의존성 주입(DI)**을 수행하는 스프링의 핵심. 개발자가 객체 생성·관리를 신경 쓰지 않고 **비즈니스 로직에 집중**하게 해준다.

---

## 1. 컨테이너의 종류

| 종류 | 설명 |
|------|------|
| **BeanFactory** | 가장 기본. 빈 생성·DI·라이프사이클, **지연 로딩** |
| **ApplicationContext** | BeanFactory 확장. 메시지 처리·이벤트·AOP 등 추가 기능 |

```java
// BeanFactory
BeanFactory beanFactory = new XmlBeanFactory(new FileSystemResource("beans.xml"));

// ApplicationContext (실무 주력)
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
MyBean myBean = (MyBean) context.getBean("myBean");
```

> 💡 실무에서는 대부분 **ApplicationContext**를 쓴다. BeanFactory의 모든 기능에 더해 메시지·이벤트·AOP 등을 제공하기 때문이다.

---

## 2. 동작 방식 3단계

```
① 빈 생성  →  ② 의존성 주입(DI)  →  ③ 라이프사이클 관리
```

| 단계 | 설명 |
|------|------|
| **① 빈 생성** | 설정(XML/어노테이션) 기반으로 빈 객체 생성 |
| **② 의존성 주입** | 빈 간 의존성을 자동 주입 → 결합도↓ |
| **③ 라이프사이클** | 초기화·소멸 메소드 호출 관리 |

```java
public class MyBean implements InitializingBean, DisposableBean {
    @Override public void afterPropertiesSet() { /* 초기화 */ }
    @Override public void destroy() { /* 소멸 */ }
}
```

---

## 3. 컨테이너 활용

| 기능 | 방법 |
|------|------|
| **프로파일** | `@Profile("dev")`로 환경별 설정 |
| **외부 설정** | `application.properties`/YAML로 분리 |
| **프로퍼티 파일** | `@PropertySource` + `Environment`로 로드 |
| **어노테이션 설정** | `@Component`, `@Service` 등으로 빈 등록 |

```java
@Profile("dev")           // 개발 환경에서만 활성화
@Configuration
public class DevConfig { ... }

@PropertySource("classpath:myapp.properties")
public class AppConfig {
    @Autowired private Environment env;
    @Bean
    public DataSource dataSource() {
        // env.getProperty("db.url") 등으로 외부 설정 사용
    }
}
```

---

## 📝 정리

```
스프링 컨테이너
├─ 역할    라이프사이클 관리 + DI
├─ 종류    BeanFactory / ApplicationContext(주력)
├─ 동작    빈 생성 → DI → 라이프사이클
└─ 활용    프로파일 · 외부 설정 · 어노테이션
```

| 개념 | 한 줄 정의 |
|------|------|
| **스프링 컨테이너** | 빈 생성·관리·주입의 핵심 |
| **ApplicationContext** | 기능이 풍부한 컨테이너 |
| **DI** | 의존성을 컨테이너가 주입 |

스프링 컨테이너는 "객체 관리를 개발자 대신 해주는 두뇌"다. 빈을 생성하고 의존성을 주입하며 생명주기를 관리하니, 개발자는 결합도 낮은 코드로 비즈니스에 집중할 수 있다.
