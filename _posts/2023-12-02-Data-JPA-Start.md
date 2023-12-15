---
title: [Spring Data JPA] Spring Data Jpa와 사용법
date: 2023-12-02 00:00:00 +0900
categories: [Database, Spring-Data-JPA]
tags: [jpa, springdatajpa]
render_with_liquid: false
future: true
---

# Spring data jpa와 사용법

## Spring data jpa?

---

기존의 JPA를 인터페이스 만으로 조작하여 사용자가 조금더 편하게 jpa를 사용할 수 있도록 만든 것.

기존의 스프링 데이터의 PagingAndSortingRepository를 상속받아 JpaRespository인터페이스를 만들었다.

## 사용방법

---

### 환경 설정

---

그래들 프로젝트에서 build.gradle 파일에 필요한 의존성들을 추가했다. 공부용으로 만들었으니 무거운 DB보다 가벼운DB(H2)를 사용하였다.

```java
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

여기서 `implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'` 는 코드를 실행할 때 insert문 등에서 입력값이 (?) 로 보이지 않는 것을 실제 들어가는 데이터를 로그단에서 볼 수 있게 해준다.

그리고 application.yml에서 적절한 환경설정 코드를 삽입한다

```java
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/datajpa
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
    # show_sql: true
        format_sql: true

logging.level:
  org.hibernate.SQL: debug
```

### 코드

---

이제 간단한 spring data jpa를 사용해보자. 먼저 jpa와 같이 엔티티가 필요하다

**Member.java**

```java
@Entity
@Getter
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;

    public Member(String username) {
        this.username = username;
    }

    public Member() {
    }
}
```

여기까진 비슷하지만 spring data jpa는 다른점은 이미 만들어진 JpaRespository인터페이스를 사용하는 방법이다.

**MemberRepository.java**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

엥? 아무것도 없는데? 맞다. 필요한 메서드들은 JpaRepository에 있으니 상속만 받고(인터페이스끼리는 extends를 사용) 사용하면 된다.

**MemberRepositoryTest.java**

```java
@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testMember(){
        Member member = new Member("memberB");
        Member savedMember = memberRepository.save(member);

        Member findMember = memberRepository.findById(savedMember.getId()).get();

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }
}
```

@SpringBootTest 어노테이션으로 테스트 파일로써 정의하고 @Transactional 어노테이션을 사용한다. 그리고 @Rollback을 false로 주어서 데이터가 삽입된 것을 눈으로 확인할 수 있도록 한다.

여기까지 사용해 보았다. 다음의 공부도 업로드 하겠다.
