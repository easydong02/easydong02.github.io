---
title: "[Spring] 스프링 컨테이너(Spring Container)"
date: 2024-02-18 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, container, bean, applicationcontext]
render_with_liquid: false
---

# **스프링 컨테이너: Spring Container**

스프링 프레임워크는 자바 기반의 애플리케이션을 개발할 때 많이 사용되는 프레임워크 중 하나입니다. 스프링의 핵심 기능 중 하나는 스프링 컨테이너입니다. 이 블로그 포스트에서는 스프링 컨테이너의 개념, 종류, 동작 방식에 대해 자세히 알아보겠습니다.

## ✅**스프링 컨테이너란?**

---

스프링 컨테이너는 스프링 프레임워크의 핵심이며, 객체의 라이프사이클을 관리하고 의존성 주입(Dependency Injection)을 수행하는 역할을 합니다. 스프링 컨테이너는 스프링 애플리케이션의 구성 요소를 생성하고 관리하여 개발자가 객체 생성과 관리에 대해 걱정하지 않고 비즈니스 로직에 집중할 수 있도록 합니다.

## ✅**스프링 컨테이너의 종류**

---

### **1. BeanFactory**

BeanFactory는 스프링의 가장 기본적인 컨테이너로서, 빈(Bean)의 생성, DI, 라이프사이클 관리 등의 기능을 제공합니다. BeanFactory는 지연 로딩을 지원하여 빈이 실제로 필요할 때 빈을 생성합니다.

```java
BeanFactory beanFactory = new XmlBeanFactory(new FileSystemResource("beans.xml"));
MyBean myBean = (MyBean) beanFactory.getBean("myBean");

```

### **2. ApplicationContext**

ApplicationContext는 BeanFactory를 확장한 인터페이스로서, BeanFactory의 모든 기능을 포함하고 있습니다. 또한 메시지 리소스 핸들링, 이벤트 발행, AOP 등 다양한 기능을 제공합니다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
MyBean myBean = (MyBean) context.getBean("myBean");

```

## ✅**스프링 컨테이너의 동작 방식**

---

### **1. 빈 생성**

스프링 컨테이너는 설정 파일(XML 또는 어노테이션)에 정의된 빈의 정보를 기반으로 빈 객체를 생성합니다.

```xml
<bean id="myBean" class="com.example.MyBean"/>

```

### **2. 의존성 주입(Dependency Injection)**

스프링 컨테이너는 빈 객체 간의 의존성을 자동으로 주입해줍니다. 이를 통해 객체 간의 결합도를 낮추고 유연한 애플리케이션을 개발할 수 있습니다.

```java
public class MyBean {
    private AnotherBean anotherBean;

    // 생성자 주입
    public MyBean(AnotherBean anotherBean) {
        this.anotherBean = anotherBean;
    }
}

```

### **3. 라이프사이클 관리**

스프링 컨테이너는 빈 객체의 라이프사이클을 관리합니다. 초기화 메서드 호출, 소멸 메서드 호출 등의 작업을 수행하여 빈 객체의 라이프사이클을 효과적으로 관리합니다.

```java
public class MyBean implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        // 빈 초기화 작업 수행
    }

    @Override
    public void destroy() throws Exception {
        // 빈 소멸 작업 수행
    }
}

```

## ✅**스프링 컨테이너의 활용**

---

### **1. 프로파일 설정**

스프링 프로파일을 사용하여 여러 환경에서 다른 설정을 적용할 수 있습니다. 특정 환경에 따라 빈의 설정을 다르게 로드하거나 활성화할 수 있습니다.

```java
javaCopy code
@Profile("dev")
@Configuration
public class DevConfig {
    // 개발 환경에 필요한 빈 설정
}

```

### **2. 외부 설정 관리**

스프링 부트의 외부 설정 관리 기능을 활용하여 애플리케이션의 설정을 외부 파일(properties 또는 YAML)로 분리하여 관리할 수 있습니다.

```
propertiesCopy code
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret

```

### **3. 프로퍼티 파일 사용**

프로퍼티 파일을 사용하여 애플리케이션의 환경변수를 쉽게 관리할 수 있습니다. 스프링 컨테이너를 통해 프로퍼티 파일을 로드하고 애플리케이션에서 사용할 수 있습니다.

```java
javaCopy code
@PropertySource("classpath:myapp.properties")
public class AppConfig {
    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(env.getProperty("db.driver"));
        dataSource.setUrl(env.getProperty("db.url"));
        dataSource.setUsername(env.getProperty("db.username"));
        dataSource.setPassword(env.getProperty("db.password"));
        return dataSource;
    }
}

```

### **4. 어노테이션 기반 설정**

XML 대신 어노테이션을 사용하여 빈의 설정을 관리할 수 있습니다. **`@Component`**, **`@Service`**, **`@Repository`**, **`@Controller`** 등의 어노테이션을 사용하여 빈을 등록하고 의존성 주입을 수행할 수 있습니다.

```java
javaCopy code
@Service
public class MyService {
    // 비즈니스 로직 구현
}

```

위와 같은 유용한 기능들을 통해 스프링 컨테이너를 더욱 효율적으로 활용할 수 있습니다. 스프링의 다양한 기능들을 활용하여 애플리케이션을 더욱 유연하고 확장 가능하게 개발할 수 있습니다.
