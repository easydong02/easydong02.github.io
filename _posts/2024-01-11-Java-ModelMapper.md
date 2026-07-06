---
title: "[Java] ModelMapper"
date: 2024-01-11 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, modelmapper]
render_with_liquid: false
---

## 📌 들어가며

계층형 애플리케이션을 짜다 보면 **엔티티(Entity) ↔ DTO** 변환 코드를 끝없이 작성하게 된다. `dto.setName(entity.getName())` 같은 반복적인 매핑은 지루하고 실수하기도 쉽다. 이 번거로움을 덜어주는 라이브러리가 **`ModelMapper`**다.

> **`ModelMapper`란?**
> Java에서 **객체 간의 매핑을 자동으로 처리**해주는 라이브러리. 주로 엔티티와 DTO 사이의 데이터 변환에 사용되어, 반복적인 매핑 코드를 최소화한다.

---

## 1. 왜 ModelMapper를 쓸까?

| 장점 | 설명 |
|------|------|
| **간편한 구성** | 별도 설정 없이 대부분의 기본 매핑 수행 → 초보자도 쉽게 사용 |
| **자동 매핑** | **필드명이 일치**하면 자동 매핑, 불일치 필드만 수동 설정 |
| **유연성** | 특정 필드의 변환 방법·제외 등을 세밀하게 제어 가능 |

```
PersonEntity  ──ModelMapper.map()──►  PersonDTO
  name, age          (필드명 일치)        name, age
                    자동으로 값 복사
```

---

## 2. 사용 예시

먼저 의존성을 추가한다. (Maven 기준)

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.4.3</version>
</dependency>
```

그다음 `map()` 메소드 한 번으로 엔티티를 DTO로 변환한다.

```java
// ModelMapper 인스턴스 생성
ModelMapper modelMapper = new ModelMapper();

// 엔티티 객체 생성
PersonEntity personEntity = new PersonEntity();
personEntity.setName("John Doe");
personEntity.setAge(25);

// DTO로 매핑 (필드명이 같으면 자동 매핑)
PersonDTO personDTO = modelMapper.map(personEntity, PersonDTO.class);

// 매핑 결과 확인
System.out.println(personDTO.getName()); // 출력: John Doe
System.out.println(personDTO.getAge());  // 출력: 25
```

`PersonEntity`(엔티티)와 `PersonDTO`(전송 객체)의 필드명이 같으므로, **`map()` 호출만으로 두 객체 간 필드가 자동 매핑**된다.

---

## 📝 정리

| 개념 | 한 줄 정의 |
|------|------|
| **ModelMapper** | 객체 간 매핑을 자동화하는 라이브러리 |
| **map()** | 원본 객체를 대상 타입으로 변환 |
| **자동 매핑 기준** | 필드명 일치 (불일치 필드는 수동 설정) |

`ModelMapper`는 "필드명이 같으면 알아서 옮겨준다"는 단순한 규칙으로 반복적인 매핑 코드를 크게 줄여준다. 엔티티-DTO 변환이 잦은 프로젝트에서 특히 유용하다.
