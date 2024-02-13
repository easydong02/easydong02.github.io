---
title: "[Spring] 커스텀 데이터 직렬화 Swagger 적용"
date: 2024-02-13 00:00:00 +0900
categories: [Backend, Spring]
tags: [spring, json, serialize]
render_with_liquid: false
---

# Custom Json Serializer : 자신만의 커스텀 직렬화를 만들어 메시지 컨버터에 장착해보자!

회사에서 제가 맡은 솔루션에 Open API문서를 자동 적용하려고 Swagger를 사용했습니다. 솔루션은 총 2개였는데 하나는 Spring boot, 그리고 나머지 하나는 그냥 Spring Framework였습니다. Spring boot 솔루션은 규모도 크지 않고 내부적으로 복잡한 설정이 없었기 때문에 검색 한대로 설정하니 바로 나왔습니다. 하지만 그냥 Spring으로 개발된 솔루션은 규모도 매우 크고 설정파일만 수십 개가 되어 적용하는데 어려움이 있었습니다. 이번 시간에는 그 문제들과 해결 방법을 적어 놓을까 합니다.

## ✅**JsonSerializer 인터페이스란?**

---

JsonSerializer 인터페이스는 Jackson 라이브러리에서 제공하는 인터페이스로, 객체를 JSON 형식으로 직렬화할 때 사용됩니다. JsonSerializer를 구현하면 특정 클래스나 타입에 대해 원하는 형태의 JSON 데이터를 생성할 수 있습니다.

## ✅**JsonSerializer 구현하기**

---

JsonSerializer를 구현하려면 다음과 같은 단계를 따라야 합니다.

1. JsonSerializer 인터페이스를 구현하는 클래스를 작성합니다.
2. serialize() 메서드를 오버라이드하여 원하는 형태의 JSON 데이터를 생성합니다.
3. 메시지 컨버터 생성에 준비한 클래스를 타입어댑터로 등록합니다.

## ✅문제 상황

---

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled.png)

스웨거를 설정하고 url로 들어갔더니 render를 할 수 없다는 페이지가 나온다. 그렇다면 실제 데이터를 확인해봅시다.

(솔루션 url)/v2/api-docs 로 접속하니 아래처럼 데이터 자체는 있긴합니다.

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled%201.png)

사진으로는 잘 안 보이는데 Json형식이지만 실상은 String으로 되어있는 것을 확인할 수 있습니다. 왜냐하면 따옴표(”)마다 역슬래시(\)가 들어가 있기 때문입니다. 화면에서 보여줄 json데이터가 없기 때문에 실제 우리가 보는 UI로 렌더링이 되지 않았던 겁니다.

✅해결 방법

---

webmvc configure를 상속받은 설정파일에 들어가 봅니다.

```java
public class RpaViewWebMvcConfiguration extends WebMvcConfigurerAdapter{

	@Override
	    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
	        converters.addAll(MessageConverterFactory.createMessageConverters());
	    }
}
```

`configureMessageConverters()` 메서드는 메시지 컨버터를 설정하는 코드를 오버라이딩으로 커스텀할 수 있습니다. 따라서 이 부분을 다시 보면, `MessageConverterFactory.createMessageConverters()` 를 추가하는 것을 알 수 있습니다. 이것은 메시지 컨버터 팩토리 클래스를 만들어서 관련 컨버터들을 생성하고 설정하는 것을 모듈화 시켰습니다.

```java
public class MessageConverterFactory {

    public static List<HttpMessageConverter<?>> createMessageConverters() {
        return Arrays.asList(
                createGsonHttpMessageConverter(),
                createStringConverter(),
                createFormHttpMessageConverter()
        );
    }
}
```

`createMessageConverters()`메서드를 좀더 들여다보면 Gson컨버터와 String컨버터와 FormHttp컨버터가 보입니다. 여기서 중요한 것은 Gson컨버터인데요. 원래 설정을 하지 않으면 Json은 `MappingJackson2HttpMessageConverter` 를 통해서 처리하지만 이 솔루션에서는 Gson컨버터를 사용하는 것을 볼 수 있습니다. 그럼 다시 `createGsonHttpMessageConverter()` 를 봅니다.

```java
public static GsonHttpMessageConverter createGsonHttpMessageConverter() {
        Gson gson = new GsonBuilder()
                .registerTypeAdapter(DateTime.class, new GsonDateTimeTypeAdapter())
                .registerTypeAdapter(String.class, new GsonStringXssTypeAdapter())
                .create();

        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        converter.setGson(gson);
        converter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON_UTF8, MediaType.APPLICATION_JSON));
        return converter;
    }
```

`createGsonHttpMessageConverter()` 메서드를 보면 Gson 빌더에서 `registerTypeAdapter()` 메서드를 통해서 각 타입에 맞는 직렬화/역직렬화 클래스를 커스텀으로 만들어서 적용하는 것을 알 수 있습니다. 저희는 swagger의 Json객체를 직렬화하는 설정을 넣어야합니다.

```java
public class GsonSpringfoxJsonSerializer implements JsonSerializer<Json> {
    @Override
    public JsonElement serialize(Json json, Type type, JsonSerializationContext context) {
        return JsonParser.parseString(json.value());
    }
}
```

이렇게 `GsonSpringfoxJsonSerializer.java`  파일을 만들고 `JsonSerializer<Json>` 를 implements합니다. 그러면 `serialize()`메서드를 오버라이딩해서 사용할 수 있습니다. 이것을 Springfox의 Json객체를 파싱할 수 있도록 설정합니다.

```java
public static GsonHttpMessageConverter createGsonHttpMessageConverter() {
        Gson gson = new GsonBuilder()
                .registerTypeAdapter(DateTime.class, new GsonDateTimeTypeAdapter())
                .registerTypeAdapter(String.class, new GsonStringXssTypeAdapter())
								.registerTypeAdapter(Json.class, new GsonSpringfoxJsonSerializer())
                .create();

        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        converter.setGson(gson);
        converter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON_UTF8, MediaType.APPLICATION_JSON));
        return converter;
    }
```

그다음 Gson빌더에 `.registerTypeAdapter(Json.class, new GsonSpringfoxJsonSerializer())` 코드를 추가하여 Springfox의 Json객체를 직렬화할 수 있도록 합니다. 

![Untitled](/assets/img/Backend/Spring/Serialize/Untitled%202.png)

이제 잘 나오는 것을 확인할 수 있습니다.