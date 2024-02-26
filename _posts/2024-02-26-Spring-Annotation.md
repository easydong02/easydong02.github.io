---
title: "[Spring] 어노테이션(Annotations)"
date: 2024-02-26 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, annotation]
render_with_liquid: false
---

## **Spring Annotation: 스프링 어노테이션 소개**

스프링 프레임워크는 다양한 어노테이션을 제공하여 개발자가 코드를 간결하게 작성할 수 있도록 지원합니다. 특히 웹 애플리케이션 개발에서는 웹MVC를 구현하기 위한 다양한 어노테이션이 존재합니다. 이번 포스팅에서는 주로 웹MVC에서 사용되는 중요한 스프링 어노테이션에 대해 알아보겠습니다.

### **1. @Controller**

**`@Controller`** 어노테이션은 스프링 MVC 컨트롤러를 정의할 때 사용됩니다. 해당 클래스가 웹 요청에 대한 처리를 담당하는 컨트롤러임을 나타냅니다.

```java
@Controller
public class MyController {
    // Controller methods
}

```

### **2. @RestController**

**`@RestController`** 어노테이션은 RESTful 웹 서비스에서 JSON 또는 XML 형식의 데이터를 반환하는데 사용됩니다. **`@Controller`** 어노테이션과 **`@ResponseBody`** 어노테이션을 합친 것과 같은 역할을 합니다.

```java
@RestController
public class MyRestController {
    // RESTful Controller methods
}

```

### **3. @RequestMapping**

**`@RequestMapping`** 어노테이션은 요청 URL과 메소드를 매핑하는데 사용됩니다. 클래스 레벨에서는 베이스 URL을 설정하고 메소드 레벨에서는 해당 URL에 대한 처리를 정의합니다.

```java
@Controller
@RequestMapping("/api")
public class MyController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello, Spring!";
    }
}

```

### **4. @RequestParam**

**`@RequestParam`** 어노테이션은 HTTP 요청 파라미터를 메소드 파라미터에 매핑하는데 사용됩니다.

```java
@GetMapping("/user")
public String getUser(@RequestParam("id") Long userId) {
    // Get user by ID
}

```

### **5. @PathVariable**

**`@PathVariable`** 어노테이션은 URL 경로의 일부를 메소드 파라미터에 매핑하는데 사용됩니다.

```java
@GetMapping("/user/{id}")
public String getUser(@PathVariable Long id) {
    // Get user by ID
}

```

### **6. @RequestBody**

**`@RequestBody`** 어노테이션은 HTTP 요청의 본문을 자바 객체로 변환하는데 사용됩니다.

```java
@PostMapping("/user")
public String addUser(@RequestBody User user) {
    // Add new user
}

```

### **7. @ResponseBody**

**`@ResponseBody`** 어노테이션은 메소드가 반환하는 객체를 HTTP 응답 본문으로 변환하는데 사용됩니다.

```java
@GetMapping("/user")
@ResponseBody
public User getUser() {
    // Return user object
}

```

### **8. @ModelAttribute**

**`@ModelAttribute`** 어노테이션은 메소드 파라미터에 바인딩된 객체를 모델에 추가하는데 사용됩니다.

```java
@GetMapping("/user")
public String getUser(@ModelAttribute("user") User user) {
    // Get user from model
}

```

위의 어노테이션들은 스프링 웹 애플리케이션을 개발할 때 자주 사용되는 중요한 어노테이션들입니다. 이 외에도 스프링 프레임워크는 다양한 어노테이션을 제공하고 있으며, 프로젝트의 요구 사항에 따라 적절한 어노테이션을 선택하여 사용할 수 있습니다.

### **9. @Autowired**

**`@Autowired`** 어노테이션은 스프링 컨테이너에서 자동으로 의존성을 주입할 때 사용됩니다. 주로 필드, 생성자, 메소드에 적용됩니다.

```java
@Controller
public class MyController {

    private final MyService myService;

    @Autowired
    public MyController(MyService myService) {
        this.myService = myService;
    }
}

```

### **10. @Service**

**`@Service`** 어노테이션은 비즈니스 로직을 수행하는 서비스 클래스에 적용됩니다. 주로 서비스 계층의 클래스에 사용됩니다.

```java
@Service
public class MyService {
    // Service methods
}

```

### **11. @Repository**

**`@Repository`** 어노테이션은 데이터 액세스 계층에서 사용되는 클래스에 적용됩니다. 주로 DAO(Data Access Object) 클래스에 사용됩니다.

```java
@Repository
public class MyRepository {
    // Repository methods
}

```

### **12. @Transactional**

**`@Transactional`** 어노테이션은 트랜잭션을 관리하는 데 사용됩니다. 메소드 또는 클래스에 적용하여 트랜잭션 범위를 설정할 수 있습니다.

```java
@Service
public class MyService {

    @Autowired
    private MyRepository myRepository;

    @Transactional
    public void saveData(Data data) {
        myRepository.save(data);
    }
}

```

### **13. @Configuration**

**`@Configuration`** 어노테이션은 스프링 빈 설정 클래스에 적용됩니다. XML 구성 파일 대신 자바 클래스에서 빈을 설정할 때 사용됩니다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

```

위의 어노테이션들은 스프링 애플리케이션을 개발할 때 자주 사용되는 유용한 어노테이션들입니다. 이외에도 스프링 프레임워크는 다양한 어노테이션을 제공하고 있으며, 프로젝트의 요구 사항에 따라 적절한 어노테이션을 선택하여 사용할 수 있습니다.
