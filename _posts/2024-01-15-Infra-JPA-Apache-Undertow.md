---
title: "[Infra] Apache vs Undertow 비교"
date: 2024-01-15 00:00:00 +0900
categories: [Infra, Webserver]
tags: [webserver, apache, undertow]
render_with_liquid: false
future: true
---

# **Apache와 Undertow: 두 웹 서버의 비교**

안녕하세요! 오늘은 웹 개발에서 자주 사용되는 두 가지 웹 서버인 Apache와 Undertow에 대해 비교해보려고 합니다. 둘 다 널리 사용되는 웹 서버이지만, 각각의 특징과 사용 시 고려해야 할 점이 있습니다.

## ✅**Apache HTTP Server**

---

### 📌**특징**

- **역사와 안정성**: Apache는 1995년부터 사용되어 온 역사를 가진 웹 서버로, 안정성과 신뢰성이 높습니다.
- **다양한 모듈 지원**: 다양한 모듈을 제공하여 기능을 확장할 수 있습니다.
- **커뮤니티 지원**: 많은 개발자들이 참여하는 커뮤니티로 인해 다양한 문제에 대한 해결책을 찾을 수 있습니다.

### **예시 코드**

Apache 서버에 적용된 가상 호스트 설정입니다.

```
apacheCopy code
<VirtualHost *:80>
    ServerName www.example.com
    DocumentRoot /var/www/html
</VirtualHost>

```

## ✅**Undertow**

---

### 📌**특징**

- **경량성과 빠른 성능**: Undertow는 경량 웹 서버로, 빠른 시작 시간과 낮은 자원 사용량을 자랑합니다.
- **Servlet 기반**: Java Servlet API를 사용하여 구현되었으며, Java 언어로 손쉽게 확장 가능합니다.
- **Embedded 모드**: Undertow는 내장된 모드로 사용될 수 있어서 별도의 웹 서버 없이도 독립적으로 실행할 수 있습니다.

### **예시 코드**

Undertow 서버의 간단한 구성 코드입니다.

```java
javaCopy code
public class SimpleUndertowServer {

    public static void main(String[] args) {
        Undertow server = Undertow.builder()
                .addHttpListener(8080, "localhost")
                .setHandler(Handlers.path()
                        .addPrefixPath("/", exchange -> exchange.getResponseSender().send("Hello, Undertow!")))
                .build();

        server.start();
    }
}

```

## ✅**어떤 것을 선택해야 할까?**

---

### 📌**Apache를 선택하는 경우**

- **큰 프로젝트와 안정성**: 큰 규모의 프로젝트에서 안정성과 신뢰성이 중요한 경우 Apache를 선택할 수 있습니다.
- **다양한 모듈 활용**: 다양한 모듈을 활용해야 하는 경우 Apache는 다양한 확장 기능을 제공합니다.

### 📌**Undertow를 선택하는 경우**

- **경량성과 빠른 시작 시간**: 경량성이 요구되거나 빠른 시작 시간이 필요한 경우 Undertow가 적합할 수 있습니다.
- **Java 언어 사용**: Java로 개발되었기 때문에 Java 언어로 손쉽게 확장하고 사용할 수 있습니다.

## ✅**결론**

---

Apache와 Undertow는 각각의 특징에 따라 선택해야 할 상황이 다릅니다. 프로젝트의 규모, 안정성의 중요성, 그리고 확장성의 필요성을 고려하여 적절한 웹 서버를 선택해보세요. 두 서버 모두 자신만의 장점을 가지고 있으므로, 상황에 맞게 선택하면 좋습니다. 즐거운 개발되세요!