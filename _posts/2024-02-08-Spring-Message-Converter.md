---
title: "[Spring] 메시지 컨버터(Message Converter)"
date: 2024-02-08 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, messageconverter]
render_with_liquid: false
---

# **Spring - MessageConverter: 메시지 변환기**

Spring 프레임워크에서는 클라이언트와 서버 간의 통신을 위해 다양한 데이터 형식을 처리할 수 있는 MessageConverter 인터페이스를 제공합니다. 이를 통해 HTTP 요청과 응답의 본문을 자바 객체로 변환하거나, 자바 객체를 다양한 형식의 메시지로 변환할 수 있습니다. 이번 글에서는 Spring의 MessageConverter에 대해 자세히 알아보고, 다양한 예시 코드를 통해 사용법을 소개하겠습니다.

## ✅**MessageConverter란?**

---

MessageConverter는 Spring MVC에서 요청 및 응답 메시지를 변환하는 인터페이스입니다. HTTP 요청과 응답의 본문을 자바 객체로 변환하거나, 자바 객체를 다양한 형식의 메시지로 변환할 수 있습니다. Spring은 이러한 변환 작업을 자동으로 처리하여 개발자가 별도의 변환 코드를 작성할 필요가 없도록 합니다.

## ✅**주요 MessageConverter 구현체**

---

Spring 프레임워크에서는 다양한 형식의 데이터를 처리하기 위한 여러 MessageConverter 구현체를 제공합니다. 대표적으로는 다음과 같은 것들이 있습니다:

- **ByteArrayHttpMessageConverter**: 바이트 배열로 변환합니다.
- **StringHttpMessageConverter**: 문자열로 변환합니다.
- **FormHttpMessageConverter**: HTML 폼 데이터로 변환합니다.
- **MappingJackson2HttpMessageConverter**: JSON 형식으로 변환합니다.
- **MarshallingHttpMessageConverter**: XML 형식으로 변환합니다.

## ✅**예시 코드**

---

### **JSON 형식으로 변환하기**

```java
import org.springframework.http.converter.json.MappingJackson2HttpMessageConverter;

@RestController
public class MyController {

    @GetMapping("/json")
    public MyObject getJson() {
        MyObject obj = new MyObject();
        obj.setId(1);
        obj.setName("John");
        return obj;
    }

    @Bean
    public MappingJackson2HttpMessageConverter jsonConverter() {
        return new MappingJackson2HttpMessageConverter();
    }
}

```

### **XML 형식으로 변환하기**

```java
import org.springframework.http.converter.xml.MarshallingHttpMessageConverter;
import org.springframework.oxm.jaxb.Jaxb2Marshaller;

@RestController
public class MyController {

    @GetMapping("/xml")
    public MyObject getXml() {
        MyObject obj = new MyObject();
        obj.setId(1);
        obj.setName("John");
        return obj;
    }

    @Bean
    public MarshallingHttpMessageConverter xmlConverter() {
        MarshallingHttpMessageConverter converter = new MarshallingHttpMessageConverter();
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setClassesToBeBound(MyObject.class);
        converter.setMarshaller(marshaller);
        converter.setUnmarshaller(marshaller);
        return converter;
    }
}

```

## ✅**사용자 정의 MessageConverter 구현**

---

Spring은 사용자가 직접 MessageConverter를 구현하여 사용할 수도 있습니다. 다음은 간단한 예시 코드입니다:

```java
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.HttpMessageNotWritableException;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.http.MediaType;
import org.springframework.http.HttpInputMessage;
import org.springframework.http.HttpOutputMessage;

public class MyCustomMessageConverter implements HttpMessageConverter<MyObject> {

    @Override
    public boolean canRead(Class<?> clazz, MediaType mediaType) {
        return false; // Implement according to your needs
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return true; // Implement according to your needs
    }

    @Override
    public List<MediaType> getSupportedMediaTypes() {
        return Collections.singletonList(MediaType.APPLICATION_JSON);
    }

    @Override
    public MyObject read(Class<? extends MyObject> clazz, HttpInputMessage inputMessage)
            throws IOException, HttpMessageNotReadableException {
        throw new UnsupportedOperationException("Not implemented");
    }

    @Override
    public void write(MyObject t, MediaType contentType, HttpOutputMessage outputMessage)
            throws IOException, HttpMessageNotWritableException {
        // Implement according to your needs
    }
}

```

## ✅**활용 팁**

---

- **다양한 형식 지원**: Spring은 다양한 형식의 데이터를 처리하기 위한 여러 MessageConverter를 제공하므로, 필요에 맞게 적절한 Converter를 선택하여 사용할 수 있습니다.
- **사용자 정의 Converter**: 필요에 따라 사용자가 직접 MessageConverter를 구현하여 커스터마이징할 수 있습니다. 이를 통해 특정한 요구사항에 맞게 데이터 변환을 처리할 수 있습니다.
- **우선순위 조정**: Spring은 여러 MessageConverter가 등록되어 있을 때 어떤 Converter를 우선적으로 사용할지 결정하기 위해 우선순위를 부여합니다. Converter의 우선순위를 조정하여 원하는 대로 동작하도록 설정할 수 있습니다.

## 📌**주의 사항**

---

- **충돌 방지**: 여러 MessageConverter가 동일한 형식의 데이터를 처리할 수 있는 경우 충돌이 발생할 수 있습니다. 이를 방지하기 위해 충돌 가능성이 있는 Converter의 우선순위를 조정하거나, 사용하지 않는 Converter를 비활성화할 수 있습니다.
- **자동 등록**: Spring은 자동으로 일부 기본 MessageConverter를 등록합니다. 따라서 사용자가 명시적으로 Converter를 등록하지 않아도 기본적인 변환 작업은 자동으로 처리됩니다.
