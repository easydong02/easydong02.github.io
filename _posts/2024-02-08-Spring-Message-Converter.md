---
title: "[Spring] 메시지 컨버터(Message Converter)"
date: 2024-02-08 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, messageconverter]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 스프링의 **MessageConverter**를 정리한다.

> **MessageConverter란?**
> Spring MVC에서 요청·응답 메시지를 변환하는 인터페이스. **HTTP 본문 ↔ 자바 객체** 변환을 담당한다. 스프링이 자동 처리하므로 개발자가 변환 코드를 따로 안 써도 된다.

```
HTTP 요청 본문 ──read──►  자바 객체
자바 객체     ──write──►  HTTP 응답 본문 (JSON/XML/...)
```

---

## 1. 주요 구현체

| 구현체 | 변환 형식 |
|------|------|
| `ByteArrayHttpMessageConverter` | 바이트 배열 |
| `StringHttpMessageConverter` | 문자열 |
| `FormHttpMessageConverter` | HTML 폼 데이터 |
| `MappingJackson2HttpMessageConverter` | **JSON** |
| `MarshallingHttpMessageConverter` | XML |

---

## 2. JSON 변환

`@RestController`에서 객체를 반환하면 **자동으로 JSON**으로 변환된다.

```java
@RestController
public class MyController {
    @GetMapping("/json")
    public MyObject getJson() {
        MyObject obj = new MyObject();
        obj.setId(1);
        obj.setName("John");
        return obj;   // → {"id":1,"name":"John"} 자동 변환
    }

    @Bean
    public MappingJackson2HttpMessageConverter jsonConverter() {
        return new MappingJackson2HttpMessageConverter();
    }
}
```

---

## 3. 사용자 정의 컨버터

필요하면 `HttpMessageConverter`를 직접 구현한다.

```java
public class MyCustomMessageConverter implements HttpMessageConverter<MyObject> {
    @Override public boolean canRead(Class<?> c, MediaType m)  { return false; }
    @Override public boolean canWrite(Class<?> c, MediaType m) { return true; }
    @Override public List<MediaType> getSupportedMediaTypes() {
        return Collections.singletonList(MediaType.APPLICATION_JSON);
    }
    @Override public MyObject read(...) { ... }
    @Override public void write(...) { ... }
}
```

| 메소드 | 역할 |
|------|------|
| `canRead` / `canWrite` | 이 타입을 변환할 수 있는가 |
| `read` / `write` | 실제 변환 |

---

## 📌 주의사항과 팁

| 항목 | 내용 |
|------|------|
| **우선순위** | 여러 컨버터 등록 시 우선순위로 선택 |
| **충돌 방지** | 같은 형식 처리 컨버터끼리 충돌 주의 |
| **자동 등록** | 기본 컨버터는 스프링이 자동 등록 |

---

## 📝 정리

```
MessageConverter
├─ 역할   HTTP 본문 ↔ 자바 객체 변환
├─ 구현체  Jackson(JSON) / String / Form / XML
├─ 커스텀  HttpMessageConverter 직접 구현
└─ 주의   우선순위 · 충돌 · 자동 등록
```

| 개념 | 한 줄 정의 |
|------|------|
| **MessageConverter** | 요청/응답 본문 변환기 |
| **MappingJackson2** | JSON 변환 (기본) |
| **커스텀 컨버터** | 특수 형식 직접 처리 |

MessageConverter는 "본문과 객체 사이의 번역기"다. `@RestController`가 객체를 JSON으로 자동 변환하는 것도 이 컨버터 덕분이다. 특수한 변환이 필요하면 직접 구현해 등록할 수 있다.
