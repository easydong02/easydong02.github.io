---
title: "[Spring] Junit4 설정 문제들(Gradle vs IntelliJ IDEA)"
date: 2024-02-23 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, gradle, intellij, junit4, test]
render_with_liquid: false
---

## 📌 들어가며

회사 프로젝트에 TDD를 적용하려 JUnit4를 설정하며 겪은 **실전 트러블슈팅** 모음이다. 여러 문제 상황과 조치, 그리고 마지막의 **진짜 원인(빌드 방식 차이)**과 해결 흐름을 정리한다.

---

## 1. 흔한 문제와 해결

| # | 문제 | 해결 |
|:--:|------|------|
| 1 | Mybatis 매퍼 인식 실패 | `@RunWith(SpringJUnit4ClassRunner.class)` + `@ContextConfiguration` |
| 2 | XML 파일 로드 실패 | 클래스패스·경로 확인 |
| 3 | 매퍼 스캔 실패 | `@MapperScan(basePackages=...)` |
| 4 | 테스트 환경 설정 실패 | 클래스패스·설정 파일 경로 확인 |
| 5 | Connection Pool 부족 | 풀 크기 증가 / `@After`에서 반환 |
| 6 | 외부 API Timeout | `@Test(timeout=5000)` / Mock |
| 7 | 환경별 테스트 데이터 | `@Profile("test")` |
| 8 | 외부 리소스 의존 | Mock 객체 / 테스트 DB |
| 9 | 동시성 문제 | 동기화 / 독립 실행 |

```java
@RunWith(SpringJUnit4ClassRunner.class)              // 스프링 컨텍스트 로드
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class MyBatisMapperTest { ... }

@Configuration
@MapperScan(basePackages = "com.example.mapper")     // 매퍼 스캔 명시
public class MyBatisConfig { ... }
```

---

## 2. ⚠️ 진짜 문제 — 빌드 방식 차이 (IntelliJ vs Gradle)

> **증상**: 같은 코드를 **정상 실행**했을 때와 **테스트 실행**했을 때, `SqlSessionFactory`에 로드된 인터페이스 리소스가 **달랐다.**

![Untitled](/assets/img/Backend/Spring/Junit-Trouble/1.png)
![Untitled](/assets/img/Backend/Spring/Junit-Trouble/2.png)

**원인은 빌드 방식의 차이**다.

| 빌드 방식 | 특징 |
|------|------|
| **IntelliJ IDEA** | **증분 빌드**(변경분만) → 빠름 |
| **Gradle** (기본) | **전체 빌드** → 정확 |

> ⚠️ **증분 빌드(incremental build)**는 변경된 부분만 빌드해 빠르지만, **"최신이다"라고 판단해 건너뛴다.** 그 결과 **이미 삭제한 파일**을 "변경 없음"으로 보고 건너뛰어, 삭제된 파일이 **그대로 포함된 채** 빌드가 완료될 수 있다. 이번 문제의 원인이 바로 이것이었다.

![Untitled](/assets/img/Backend/Spring/Junit-Trouble/3.png)

> 💡 **해결**: 정확한 빌드가 필요한 **테스트 코드는 Gradle 빌드**로, 빠른 반복이 필요한 **정상 실행은 IntelliJ IDEA 빌드**로 나눠 설정했다.

---

## 📝 정리

```
JUnit4 트러블슈팅
├─ 흔한 문제  @RunWith / @MapperScan / @Profile / Mock 등
└─ 진짜 원인  빌드 방식 차이
   ├─ IntelliJ = 증분 빌드(빠름, 삭제 파일 잔존 위험)
   └─ Gradle = 전체 빌드(정확)
   → 테스트는 Gradle, 실행은 IntelliJ
```

| 개념 | 한 줄 정의 |
|------|------|
| **증분 빌드** | 변경분만 빌드(빠름, 부정확) |
| **전체 빌드** | 전부 빌드(느림, 정확) |
| **@RunWith** | 스프링 컨텍스트로 테스트 실행 |

이 트러블슈팅의 핵심 교훈은, **"테스트가 이상하게 동작하면 빌드 방식을 의심하라"**는 것이다. 증분 빌드는 빠르지만 삭제된 파일을 놓칠 수 있으므로, 정확성이 중요한 테스트는 Gradle 전체 빌드로 검증하는 것이 안전하다.
