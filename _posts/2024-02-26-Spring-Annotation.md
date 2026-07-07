---
title: "[Spring] 어노테이션(Annotations)"
date: 2024-02-26 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, annotation]
render_with_liquid: false
---

## 📌 들어가며

스프링은 다양한 **어노테이션**으로 코드를 간결하게 만든다. 이번 글에서는 특히 **웹 MVC**와 **빈 관리**에서 자주 쓰이는 핵심 어노테이션을 정리한다.

---

## 1. 웹 MVC 어노테이션

| 어노테이션 | 역할 |
|------|------|
| `@Controller` | MVC 컨트롤러 정의 |
| `@RestController` | `@Controller` + `@ResponseBody` (JSON/XML 반환) |
| `@RequestMapping` | URL ↔ 메소드 매핑 |
| `@RequestParam` | 요청 파라미터 → 메소드 파라미터 |
| `@PathVariable` | URL 경로 일부 → 메소드 파라미터 |
| `@RequestBody` | 요청 본문 → 자바 객체 |
| `@ResponseBody` | 반환 객체 → 응답 본문 |
| `@ModelAttribute` | 바인딩 객체를 모델에 추가 |

```java
@RestController
@RequestMapping("/api")
public class MyController {

    @GetMapping("/user/{id}")           // URL 경로
    public User getUser(@PathVariable Long id) { ... }

    @GetMapping("/search")
    public String search(@RequestParam("q") String query) { ... }  // ?q=...

    @PostMapping("/user")
    public String addUser(@RequestBody User user) { ... }  // 본문 → 객체
}
```

> 💡 `@RestController`는 REST API의 표준이다. `@Controller`에 `@ResponseBody`를 매 메소드에 붙이는 대신, 클래스에 하나만 붙이면 된다.

---

## 2. 빈 등록·의존성 주입 어노테이션

| 어노테이션 | 역할 | 계층 |
|------|------|------|
| `@Component` | 일반 빈 등록 | - |
| `@Controller` | 프레젠테이션 계층 | 웹 |
| `@Service` | 비즈니스 로직 | 서비스 |
| `@Repository` | 데이터 액세스 | DAO |
| `@Autowired` | **의존성 자동 주입** | - |
| `@Configuration` | 자바 기반 빈 설정 | - |
| `@Transactional` | 트랜잭션 관리 | - |

```java
@Service
public class MyService {
    private final MyRepository myRepository;

    @Autowired   // 생성자 주입 (권장)
    public MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }

    @Transactional   // 트랜잭션 범위 설정
    public void saveData(Data data) { myRepository.save(data); }
}
```

> 💡 `@Controller`·`@Service`·`@Repository`는 모두 **`@Component`의 특수화**로, 계층을 명확히 드러내는 역할도 한다.

---

## 📝 정리

```
Spring 어노테이션
├─ 매핑     @Controller/@RestController + @RequestMapping
├─ 파라미터  @RequestParam / @PathVariable / @RequestBody
├─ 빈 등록  @Component / @Service / @Repository
├─ 주입     @Autowired (생성자 권장)
└─ 기타     @Configuration, @Transactional
```

| 개념 | 한 줄 정의 |
|------|------|
| **@RestController** | JSON 반환 컨트롤러 |
| **@Autowired** | 의존성 자동 주입 |
| **@Service/@Repository** | 계층별 빈 등록 |

스프링 어노테이션은 "무엇을 하는 클래스인지"를 선언적으로 드러낸다. 웹 MVC 매핑과 빈 등록·주입 어노테이션만 확실히 잡으면, 대부분의 스프링 애플리케이션을 읽고 쓸 수 있다.
