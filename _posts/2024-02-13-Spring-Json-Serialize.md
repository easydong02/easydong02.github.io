---
title: "[Spring] 커스텀 데이터 직렬화 Swagger 적용"
date: 2024-02-13 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, json, serialize]
render_with_liquid: false
---

## 📌 들어가며

회사에서 맡은 솔루션에 Swagger로 Open API 문서를 적용하다 만난 **실전 트러블슈팅** 기록이다. Spring Boot 솔루션은 검색대로 하니 바로 됐지만, **순수 Spring**으로 개발된 대형 솔루션(설정 파일 수십 개)에서는 문제가 생겼다. **커스텀 JSON 직렬화**로 이를 해결한 과정을 정리한다.

> **JsonSerializer란?** Jackson(또는 Gson)에서 객체를 JSON으로 직렬화할 때 쓰는 인터페이스. 구현하면 특정 타입에 대해 **원하는 형태의 JSON**을 만들 수 있다.

---

## 1. 문제 상황

Swagger URL로 들어가니 **"render를 할 수 없다"**는 페이지가 나온다.

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled.png)

`(url)/v2/api-docs`로 실제 데이터를 확인해보니, 데이터는 있지만...

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled%201.png)

> ⚠️ **원인**: JSON 형식이지만 실상은 **String**이었다. 따옴표(`"`)마다 **역슬래시(`\`)**가 붙어 있었기 때문. 화면에 보여줄 진짜 JSON이 아니라 문자열이라 렌더링이 안 된 것이다.

---

## 2. 원인 추적 — Gson 컨버터

WebMvc 설정을 보니, 이 솔루션은 JSON을 (기본 `MappingJackson2` 대신) **Gson 컨버터**로 처리하고 있었다.

```java
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    converters.addAll(MessageConverterFactory.createMessageConverters());
}
```

```java
public static GsonHttpMessageConverter createGsonHttpMessageConverter() {
    Gson gson = new GsonBuilder()
        .registerTypeAdapter(DateTime.class, new GsonDateTimeTypeAdapter())
        .registerTypeAdapter(String.class, new GsonStringXssTypeAdapter())
        .create();
    // ...
}
```

> 💡 `registerTypeAdapter()`로 **타입별 커스텀 직렬화**를 등록하고 있었다. 여기에 **Swagger의 Json 객체를 위한 직렬화**를 추가하면 된다.

---

## 3. 해결 — 커스텀 JsonSerializer 등록

Springfox의 `Json` 객체를 파싱하는 직렬화 클래스를 만든다.

```java
public class GsonSpringfoxJsonSerializer implements JsonSerializer<Json> {
    @Override
    public JsonElement serialize(Json json, Type type, JsonSerializationContext context) {
        return JsonParser.parseString(json.value());   // 문자열이 아닌 실제 JSON으로 파싱
    }
}
```

그리고 Gson 빌더에 **등록**한다.

```java
Gson gson = new GsonBuilder()
    .registerTypeAdapter(DateTime.class, new GsonDateTimeTypeAdapter())
    .registerTypeAdapter(String.class, new GsonStringXssTypeAdapter())
    .registerTypeAdapter(Json.class, new GsonSpringfoxJsonSerializer())  // ★ 추가
    .create();
```

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled%202.png)

이제 Swagger UI가 정상적으로 렌더링된다.

---

## 📝 정리

```
커스텀 JSON 직렬화 (트러블슈팅)
├─ 증상   Swagger UI 렌더 불가 (JSON이 String으로, \" 형태)
├─ 원인   Gson 컨버터가 Springfox Json을 문자열로 직렬화
├─ 해결   JsonSerializer<Json> 구현 → JsonParser로 실제 JSON 파싱
└─ 등록   GsonBuilder.registerTypeAdapter(Json.class, ...)
```

| 개념 | 한 줄 정의 |
|------|------|
| **JsonSerializer** | 타입별 커스텀 직렬화 |
| **registerTypeAdapter** | Gson에 타입별 어댑터 등록 |
| **문제 본질** | JSON이 String으로 이중 직렬화됨 |

이 트러블슈팅의 교훈은, **직렬화 방식(Jackson vs Gson)과 커스텀 타입 어댑터**를 이해해야 실전 문제를 풀 수 있다는 것이다. 대형 레거시 솔루션일수록 이런 세부 설정을 추적하는 능력이 중요하다.
