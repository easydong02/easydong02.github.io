---
title: "[Spring] Junit4 설정 문제들(Gradle vs IntelliJ IDEA)"
date: 2024-02-23 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, gradle, intellij, junit4, test]
render_with_liquid: false
---

# **JUnit4 여러 문제 상황들과 해결 방법**

최근에 회사 프로젝트에 TDD 적용을 위해 Junit설정을 했습니다. JUnit은 자바 언어용 유닛 테스트 프레임워크 중 하나로, 개발자가 자신이 작성한 코드를 테스트할 수 있게 도와줍니다. 하지만 JUnit을 사용하다보면 다양한 문제 상황에 직면할 수 있습니다. 이번 글에서는 제가 JUnit4를 적용하는 과정에 문제가 생겨서 제가 했던 조치들과 마지막에는 진짜 문제와 해결 흐름을 적었습니다.

## **1. Mybatis 매퍼 인식 실패**

Mybatis를 사용하여 데이터베이스와 상호 작용하는 테스트를 작성할 때, 매퍼 인터페이스를 인식하지 못하는 경우가 있습니다. 이는 JUnit이 Mybatis의 매퍼 인터페이스를 올바르게 스캔하지 못하는 문제일 수 있습니다.

**해결 방법:** **`@RunWith(SpringJUnit4ClassRunner.class)`** 어노테이션을 사용하여 JUnit이 스프링 컨텍스트를 올바르게 로드하도록 설정합니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class MyBatisMapperTest {
    // 테스트 코드 작성
}
```

## **2. XML 파일 로드 실패**

JUnit 테스트에서 XML 파일을 로드할 때 파일을 찾지 못하는 경우가 있습니다. 이는 클래스패스 설정이 올바르지 않거나 파일 경로가 잘못된 경우입니다.

**해결 방법:** 클래스패스의 위치를 확인하고, 올바른 경로에 XML 파일이 있는지 확인합니다.

```java
@ContextConfiguration(locations = {"classpath:test-context.xml"})
public class XmlLoadingTest {
    // 테스트 코드 작성
}
```

## **3. 매퍼 스캔 실패**

스프링이 Mybatis 매퍼를 스캔하지 못하는 경우가 있습니다. 이는 스캔할 패키지를 정확하게 지정하지 않거나 Mybatis 설정이 올바르지 않은 경우입니다.

**해결 방법:** **`@MapperScan`** 어노테이션을 사용하여 Mybatis 매퍼를 스캔할 패키지를 명시적으로 지정합니다.

```java
@Configuration
@MapperScan(basePackages = "com.example.mapper")
public class MyBatisConfig {
    // Mybatis 설정
}
```

## **4. 테스트 환경 설정 실패**

JUnit 테스트가 실행되지 않고 설정 파일을 찾지 못하는 경우가 있습니다. 이는 클래스패스 설정이 잘못된 경우나 테스트 환경이 올바르게 구성되지 않은 경우일 수 있습니다.

**해결 방법:** 클래스패스 설정을 다시 확인하고, 스프링 설정 파일이나 프로퍼티 파일의 경로가 올바른지 확인합니다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:test-context.xml"})
public class MyTest {
    // 테스트 코드 작성
}
```

## **5. 데이터베이스 연동 테스트 시 Connection Pool 부족**

**상황:** 데이터베이스 연동 테스트를 실행할 때 Connection Pool이 부족하여 테스트가 실패하는 경우가 있습니다.

**해결 방법:** 테스트용 데이터베이스 연결 설정에서 Connection Pool의 크기를 증가시키거나, 테스트가 끝나면 사용한 Connection을 명시적으로 반환하여 Connection Pool을 재사용할 수 있도록 합니다.

```java
@Before
public void setUp() throws Exception {
    // Connection Pool 설정
}

@After
public void tearDown() throws Exception {
    // 사용한 Connection 반환
}
```

## **6. 외부 API 호출 시 Timeout 발생**

**상황:** 외부 API를 호출하는 테스트를 실행할 때 Timeout이 발생하여 테스트가 실패하는 경우가 있습니다.

**해결 방법:** 외부 API 호출 시 Timeout 설정을 늘리거나, 대안으로 모의 객체(Mock Object)를 사용하여 외부 의존성을 제어할 수 있습니다.

```java
@Test(timeout = 5000) // 5초 Timeout 설정
public void testExternalAPICall() {
    // 외부 API 호출
}
```

## **7. 환경에 따른 테스트 데이터 설정**

**상황:** 로컬, 개발, 테스트 환경에 따라 테스트 데이터가 달라지는 경우가 있습니다.

**해결 방법:** 테스트 프로파일을 설정하여 각 환경에 맞는 데이터를 로드하도록 합니다.

```java
@Profile("test")
@Configuration
public class TestConfig {
    // 테스트용 데이터 설정
}
```

## **8. 외부 리소스에 의존하는 테스트**

**상황:** 테스트 코드가 외부 리소스에 의존하여 실행이 불안정한 경우가 있습니다.

**해결 방법:** 모의 객체(Mock Object)를 사용하여 외부 리소스의 동작을 시뮬레이션하거나, 테스트용 데이터베이스를 사용하여 외부 의존성을 최소화합니다.

```java
@Test
public void testExternalResource() {
    // 외부 리소스를 모의 객체로 대체하여 테스트
}
```

## **9. 동시성 문제 발생**

**상황:** 동시에 실행되는 테스트 간에 상태 공유로 인한 동시성 문제가 발생하는 경우가 있습니다.

**해결 방법:** 동기화(Synchronization)를 사용하여 상태 공유를 방지하거나, 테스트 간에 상태를 공유하지 않도록 각 테스트 메소드가 독립적으로 실행되도록 합니다.

```java
@Test
public synchronized void testConcurrencyIssue() {
    // 동기화하여 동시성 문제 방지
}
```

위의 상황과 해결 방법을 참고하여 JUnit4 테스트 코드를 보다 안정적으로 작성할 수 있습니다.

## 10. 빌드 문제(IntelliJ IDEA vs Gradle)

상황: 디버깅을 하다가 같은 코드를 정상 실행했을 때와 테스트 코드를 실행했을 때의 SqlSessionFactory에 있는 로드된 인터페이스 리소스들이 다른 것을 확인했습니다.

Intellij IDEA로 테스트코드 실행했을 때:

![Untitled](/assets/img/Backend/Spring/Junit-Trouble/1.png)

Intellij IDEA로 코드 정상 실행했을 때:

![Untitled](/assets/img/Backend/Spring/Junit-Trouble/2.png)

### 이유

이유는 바로 빌드 방식의 차이입니다. IntelliJ 내부의 빌드 방식은 증분빌드임에 비해 디폴트인 Gradle 빌드방식은 전체 빌드인거죠!

### 증분빌드(incremental build)?

 증분 빌드는 용어 그대로 증분된 부분, 즉 변경된 부분만 빌드를 하는 방식으로 변경되지 않은 것에 대해서는 건너뛰고 빌드를 진행해서 빠른 빌드를 원할 경우 선택하는 방법입니다. **IntelliJ IDEA가 바로 증분 빌드이고요.** 그래서 IntelliJ IDEA가 Gradle 빌드 방식보다 빠르게 빌드를 수행할 수 있었던 것입니다

 하지만 증분 빌드이므로 **`IntelliJ IDEA 빌드 방식은 최신이다`라고 판단하고 건너뛰고 빌드를 진행해** 이번에 겪었던 것과 같이 이미 삭제한 파일에 대해서 변경 사항이 없다라고 판단해 건너뛰고 빌드를 진행해 빌드를 진행하고 나온 결과물에 **삭제됐던 파일이 그대로 포함된 상태로 빌드가 완료될 수 있습니다.** 따라서 정확한 빌드를 원한다면 IntelliJ IDEA 빌드 방식이 아닌 Gradle 빌드 방식을 선택해야하는 것입니다.

![Untitled](/assets/img/Backend/Spring/Junit-Trouble/3.png)

그래서 이런식으로 테스트 코드를 빌드할때는 Gradle로 하고 정상실행은 intelliJ IDEA로 했습니다!