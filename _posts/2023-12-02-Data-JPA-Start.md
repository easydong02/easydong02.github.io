---
title: "[Spring Data JPA] Spring Data Jpa와 사용법"
date: 2023-12-02 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [jpa, springdatajpa]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **Spring Data JPA**의 개념과 기본 설정·사용법을 정리한다.

> **Spring Data JPA란?**
> 기존 JPA를 **인터페이스만으로** 조작하게 해, 더 편하게 쓸 수 있도록 만든 것. 스프링 데이터의 `PagingAndSortingRepository`를 상속해 `JpaRepository` 인터페이스를 제공한다.

---

## 1. 환경 설정

**build.gradle** — 공부용이라 가벼운 H2 DB를 사용한다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
}
```

> 💡 **p6spy**는 실행 시 insert문 등에서 `?`로 가려지는 **실제 입력값을 로그에서 보이게** 해준다.

**application.yml:**

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
logging.level:
  org.hibernate.SQL: debug
```

---

## 2. 코드 — 엔티티 + Repository

**Member.java** (엔티티):

```java
@Entity
@Getter
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;

    public Member(String username) { this.username = username; }
    public Member() {}
}
```

**MemberRepository.java** — 여기가 핵심이다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

> 💡 **내용이 비어 있는데?** 맞다. 필요한 메소드(`save`, `findById` 등)는 이미 `JpaRepository`에 있으므로, **상속만 받으면**(인터페이스끼리는 `extends`) 바로 쓸 수 있다.

---

## 3. 테스트

```java
@SpringBootTest
@Transactional
@Rollback(value = false)   // 데이터 삽입을 눈으로 확인
class MemberRepositoryTest {
    @Autowired MemberRepository memberRepository;

    @Test
    public void testMember() {
        Member member = new Member("memberB");
        Member savedMember = memberRepository.save(member);      // 저장

        Member findMember = memberRepository.findById(savedMember.getId()).get();

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember).isEqualTo(member);
    }
}
```

| 어노테이션 | 역할 |
|------|------|
| `@SpringBootTest` | 통합 테스트 |
| `@Transactional` | 테스트 후 롤백 |
| `@Rollback(false)` | 롤백 끄기(데이터 확인용) |

---

## 📝 정리

```
Spring Data JPA 시작
├─ 개념   JPA를 인터페이스만으로 사용 (JpaRepository)
├─ 설정   build.gradle + application.yml (H2, p6spy)
├─ 코드   엔티티 + JpaRepository 상속만
└─ 테스트  @SpringBootTest + @Transactional
```

| 개념 | 한 줄 정의 |
|------|------|
| **JpaRepository** | 기본 CRUD를 제공하는 인터페이스 |
| **p6spy** | SQL 파라미터를 로그로 확인 |
| **extends만** | 상속만으로 CRUD 사용 |

Spring Data JPA의 매력은 **"인터페이스를 상속만 하면 CRUD가 끝"**이라는 것이다. 반복적인 Repository 코드를 없애주니, 개발자는 비즈니스 로직에 집중할 수 있다.
