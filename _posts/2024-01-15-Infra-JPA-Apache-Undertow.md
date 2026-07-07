---
title: "[Infra] Apache vs Undertow 비교"
date: 2024-01-15 00:00:00 +0900
categories: [Infra, Webserver]
tags: [webserver, apache, undertow]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 웹 개발에서 자주 쓰이는 두 웹 서버 **Apache**와 **Undertow**를 비교한다. 둘 다 널리 쓰이지만 성격이 다르다.

| 서버 | 한 줄 성격 |
|------|------|
| **Apache HTTP Server** | 역사·안정성·다양한 모듈 |
| **Undertow** | 경량·빠른 성능·Java 내장 |

---

## 1. Apache HTTP Server

| 특징 | 설명 |
|------|------|
| **역사·안정성** | 1995년부터, 높은 신뢰성 |
| **다양한 모듈** | 모듈로 기능 확장 |
| **커뮤니티** | 방대한 커뮤니티 지원 |

```apache
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot /var/www/html
</VirtualHost>
```

## 2. Undertow

| 특징 | 설명 |
|------|------|
| **경량·고성능** | 빠른 시작, 낮은 자원 사용 |
| **Servlet 기반** | Java Servlet API로 구현, Java로 확장 |
| **Embedded 모드** | 별도 웹 서버 없이 독립 실행 |

```java
public class SimpleUndertowServer {
    public static void main(String[] args) {
        Undertow server = Undertow.builder()
            .addHttpListener(8080, "localhost")
            .setHandler(Handlers.path()
                .addPrefixPath("/", exchange ->
                    exchange.getResponseSender().send("Hello, Undertow!")))
            .build();
        server.start();
    }
}
```

> 💡 Undertow는 **내장(Embedded)** 모드가 강점이다. 스프링 부트에서 톰캣 대신 Undertow를 내장 서버로 쓰기도 한다.

---

## 3. 어떤 것을 선택할까?

| 선택 | 상황 |
|------|------|
| **Apache** | 큰 규모, 안정성 중시, 다양한 모듈 필요 |
| **Undertow** | 경량성·빠른 시작, Java 기반 확장 |

---

## 📝 정리

```
Apache vs Undertow
├─ Apache    안정성·모듈·커뮤니티 (전통 웹 서버)
├─ Undertow  경량·빠름·Java 내장 (Embedded)
└─ 선택      규모·안정성 vs 경량성·Java
```

| 개념 | 한 줄 정의 |
|------|------|
| **Apache** | 역사 깊은 안정적 웹 서버 |
| **Undertow** | 경량 Java 내장 웹 서버 |
| **Embedded 모드** | 앱에 내장해 독립 실행 |

웹 서버 선택은 정답이 없다. **규모와 안정성이 중요하면 Apache**, **경량성과 Java 통합이 중요하면 Undertow**다. 프로젝트의 요구사항에 맞게 고르면 된다.
