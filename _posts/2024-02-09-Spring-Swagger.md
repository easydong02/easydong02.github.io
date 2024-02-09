---
title: "[Spring] 스웨거로 API 문서 자동화(Swagger)"
date: 2024-02-09 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, swagger]
render_with_liquid: false
---

# **Spring - Swagger: API 문서 자동화**

Spring 프레임워크에서는 Swagger를 통해 API 문서를 자동으로 생성하고 관리할 수 있습니다. Swagger를 사용하여 개발자는 API의 명세를 쉽게 확인하고 테스트할 수 있으며, API의 이해도를 높이고 개발 생산성을 향상시킬 수 있습니다. 이번 글에서는 Swagger를 사용하여 Spring 애플리케이션의 API 문서를 자동화하는 방법에 대해 알아보겠습니다.

## ✅**Swagger란?**

---

Swagger는 API를 설계, 빌드, 문서화하고, 개발자들이 쉽게 사용할 수 있도록 하는 도구입니다. Swagger는 OpenAPI Specification(OAS)라는 표준을 기반으로 작동하며, API를 자동으로 문서화하는 기능을 제공합니다. 이를 통해 개발자는 API에 대한 이해도를 높이고, API를 쉽게 테스트하고 사용할 수 있습니다.

## ✅**Swagger 설정하기**

---

Spring 프로젝트에서는 Swagger를 사용하기 위해서는 먼저 의존성을 추가해야 합니다. Maven을 사용하는 경우, **`pom.xml`** 파일에 다음과 같이 의존성을 추가할 수 있습니다:

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>

```

그리고 Swagger 설정을 위한 **`@Configuration`** 클래스를 작성합니다:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
          .select()
          .apis(RequestHandlerSelectors.basePackage("com.example.controller"))
          .paths(PathSelectors.any())
          .build();
    }
}

```

위의 설정에서 **`basePackage`**는 API 컨트롤러 클래스가 위치한 패키지를 지정합니다.

## ✅**Swagger UI 확인하기**

---

Swagger UI를 확인하려면 애플리케이션을 실행한 후 브라우저에서 **`http://localhost:8080/swagger-ui.html`**에 접속합니다. 이렇게 하면 API 명세서가 자동으로 생성되어 표시됩니다.

## ✅**주요 기능**

---

Swagger를 사용하면 다음과 같은 주요 기능을 활용할 수 있습니다:

- **API 문서 자동 생성**: Swagger를 통해 API의 자동 문서화를 수행할 수 있습니다.
- **API 테스트**: Swagger UI를 통해 API를 직접 테스트하고 결과를 확인할 수 있습니다.
- **API 버전 관리**: 다양한 API 버전을 관리하고 문서화할 수 있습니다.

## ✅**사용 예시**

---

다음은 간단한 Spring Boot 애플리케이션에서 Swagger를 사용하는 예시 코드입니다:
