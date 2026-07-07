---
title: "[Junit] Junit을 활용한 테스트 코드 작성"
date: 2023-12-20 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, junit]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 자바 개발자가 가장 많이 쓰는 테스트 프레임워크 **JUnit**을 정리한다.

> **JUnit이란?** 자바용 유닛 테스트 프레임워크(자바 개발자의 93%가 사용). 자바 8+·스프링부트 2.2+에서는 **JUnit5**를 쓴다.

**JUnit5의 구성:**

```
JUnit5 = Platform + Jupiter(5) + Vintage(3,4)
```

| 구성 | 역할 |
|------|------|
| **Platform** | 테스트 실행 런처(TestEngine API) |
| **Jupiter** | JUnit5용 TestEngine 구현체 |
| **Vintage** | JUnit 3·4 지원 구현체 |

> 💡 **스프링 테스트 = JUnit(프레임워크) + Assertions(메소드)**. `assertThat` 같은 단정 메소드는 AssertJ가 제공한다.

![Desktop View](/assets/img/Backend/Spring/Junit/1.png)

---

## 1. 기본 어노테이션

| 어노테이션 | 역할 |
|------|------|
| `@Test` | 테스트 메소드 |
| `@DisplayName` | 표시 이름(한글·이모지 가능) |
| `@BeforeAll` / `@AfterAll` | 클래스 전체 전/후 **한 번** |
| `@BeforeEach` / `@AfterEach` | 각 테스트 전/후 |
| `@Disabled` | 비활성화 |
| `@RepeatedTest(n)` | n번 반복 |
| `@ParameterizedTest` | 여러 값으로 반복 |

```java
@Test
void 회원가입() {
    // given - when - then 패턴 추천
    Member member = new Member();
    member.setName("spring");                          // given

    Long saveId = memberService.join(member);          // when

    Member findMember = memberService.findOne(saveId).get();  // then
    assertThat(member.getName()).isEqualTo(findMember.getName());
}
```

```java
@AfterEach
public void afterEach() { memberRepository.clearStore(); }   // 매 테스트 후 초기화

@ParameterizedTest
@ValueSource(ints = {1, 3, 5, -3, 15})   // 여러 값으로 반복
void isOdd(int number) { assertTrue(number % 2 == 1); }
```

> 💡 **given-when-then** 패턴(준비-실행-검증)으로 작성하면 테스트 의도가 명확해진다.

---

## 2. Assertions 메소드

| 메소드 | 역할 |
|------|------|
| `assertThat(a).isEqualTo(b)` | 값 비교 (AssertJ, 체이닝) |
| `assertEquals(a, b)` | 값이 같은지 |
| `assertSame(a, b)` | 같은 객체인지 |
| `assertAll(...)` | 여러 단정을 모두 실행 후 검증 |
| `assertThrows(예외, 실행)` | 예외 발생 검증 |
| `assumeTrue` / `assumingThat` | 조건 만족 시에만 테스트 |

**예외 테스트:**

```java
@Test
public void 중복_회원_예외() {
    Member member = new Member();  member.setName("spring");
    Member member2 = new Member(); member2.setName("spring");
    memberService.join(member);

    // 예외 발생을 검증
    IllegalStateException e = assertThrows(IllegalStateException.class,
        () -> memberService.join(member2));
    assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
}
```

**assertAll** — 여러 단정을 모두 실행:

```java
@Test
void 계산() {
    assertAll(
        () -> assertEquals(2, 1 - 1),   // 실패
        () -> assertEquals(0, 1 - 1)    // 성공
    );   // 둘 다 실행 후 결과 종합
}
```

> 💡 `assumeTrue`는 조건이 false면 **테스트 전체**를 건너뛰고, `assumingThat`은 **전달된 코드 블록만** 건너뛴다.

---

## 📝 정리

```
JUnit
├─ 구성       Platform + Jupiter + Vintage
├─ 어노테이션  @Test / @BeforeEach / @ParameterizedTest 등
├─ 패턴       given-when-then
└─ 단정       assertThat / assertEquals / assertThrows / assertAll
```

| 개념 | 한 줄 정의 |
|------|------|
| **@Test** | 테스트 메소드 표시 |
| **given-when-then** | 준비-실행-검증 |
| **assertThat** | 값 검증(체이닝) |
| **assertThrows** | 예외 검증 |

JUnit은 "코드가 의도대로 동작하는지 자동 검증"하는 도구다. 어노테이션으로 생명주기를 관리하고, `assertThat`·`assertThrows` 등으로 결과를 단정하면, 안정적인 코드를 지속적으로 유지할 수 있다.
