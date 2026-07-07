---
title: "[Spring] 스웨거로 API 문서 자동화(Swagger)"
date: 2024-02-09 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, swagger]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **Swagger**로 Spring 애플리케이션의 API 문서를 자동화하는 방법을 정리한다.

> **Swagger란?**
> API를 **설계·빌드·문서화**하고 개발자가 쉽게 사용하도록 돕는 도구. **OpenAPI Specification(OAS)** 표준 기반으로, API를 자동 문서화한다. 문서를 손으로 관리하는 수고를 없애준다.

---

## 1. 설정

**의존성 추가** (Maven):

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

**설정 클래스** (`@Configuration`):

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.example.controller"))  // 컨트롤러 패키지
            .paths(PathSelectors.any())
            .build();
    }
}
```

> 💡 `basePackage`에 **API 컨트롤러가 위치한 패키지**를 지정하면, 그 안의 API가 자동으로 문서화된다.

---

## 2. Swagger UI 확인

애플리케이션 실행 후 브라우저에서 접속한다.

```
http://localhost:8080/swagger-ui.html
```

API 명세서가 자동 생성되어 표시된다.

---

## 3. 주요 기능

| 기능 | 설명 |
|------|------|
| **API 문서 자동 생성** | 코드에서 명세를 자동 추출 |
| **API 테스트** | Swagger UI에서 직접 호출·결과 확인 |
| **버전 관리** | 여러 API 버전 문서화 |

> 💡 Swagger UI의 강점은 **문서 안에서 바로 API를 테스트**할 수 있다는 것이다. 프론트엔드·백엔드 협업 시 문서와 테스트를 한곳에서 해결한다.

---

## 📝 정리

```
Spring Swagger
├─ 개념   OAS 기반 API 자동 문서화
├─ 설정   springfox 의존성 + @EnableSwagger2 + Docket
├─ 확인   /swagger-ui.html
└─ 기능   자동 문서 + 테스트 + 버전 관리
```

| 개념 | 한 줄 정의 |
|------|------|
| **Swagger** | API 자동 문서화 도구 |
| **Docket** | Swagger 설정 빈 |
| **Swagger UI** | 문서 + 테스트 웹 화면 |

Swagger는 "코드가 곧 문서"를 실현한다. 컨트롤러만 만들면 API 문서가 자동 생성·갱신되니, 문서 관리 부담을 크게 줄이고 협업 생산성을 높인다.
