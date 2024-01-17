---
title: "[Java] ModelMapper"
date: 2024-01-11 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, modelmapper]
render_with_liquid: false
---

# ModelMapper

## ✅**ModelMapper: 객체 간 편리한 매핑을 위한 도구**

---

안녕하세요! 오늘은 개발자들 사이에서 많이 사용되는 **`ModelMapper`**에 대해 알아보겠습니다. 이 라이브러리는 엔티티(Entity)와 데이터 전송 객체(DTO) 간의 변환 작업을 간편하게 처리해주는 도구로, 번거로운 매핑 작업을 효과적으로 해결할 수 있습니다.

## ✅**ModelMapper란?**

---

**`ModelMapper`**는 Java에서 객체 간의 매핑을 자동으로 처리해주는 라이브러리입니다. 주로 엔티티와 DTO 사이에서 데이터를 전환할 때 사용되며, 이를 통해 반복적이고 지루한 매핑 코드 작성을 최소화할 수 있습니다.

## ✅**왜 ModelMapper를 사용해야 할까?**

---

1. **간편한 구성**: ModelMapper는 간단한 구성만으로도 사용할 수 있습니다. 별도의 설정 없이도 대부분의 기본 매핑 작업을 수행할 수 있어 초보자들도 쉽게 활용할 수 있습니다.
2. **자동 매핑**: 필드명이 일치하는 경우 자동으로 매핑해줍니다. 일치하지 않는 필드는 수동으로 설정할 수 있습니다.
3. **유연성**: 매핑 규칙을 세부적으로 설정할 수 있어, 특정 필드의 변환 방법이나 제외할 필드 등을 세밀하게 제어할 수 있습니다.

## 📌**사용 예시**

---

자, 실제로 **`ModelMapper`**를 어떻게 사용하는지 예시 코드를 통해 알아보겠습니다.

```java

// 의존성 추가 (Maven 기준)
// pom.xml
// <dependency>
//     <groupId>org.modelmapper</groupId>
//     <artifactId>modelmapper</artifactId>
//     <version>2.4.3</version>
// </dependency>

// ModelMapper 인스턴스 생성
ModelMapper modelMapper = new ModelMapper();

// 엔티티 객체 생성
PersonEntity personEntity = new PersonEntity();
personEntity.setName("John Doe");
personEntity.setAge(25);

// DTO로 매핑
PersonDTO personDTO = modelMapper.map(personEntity, PersonDTO.class);

// 매핑 결과 확인
System.out.println(personDTO.getName()); // 출력: John Doe
System.out.println(personDTO.getAge());  // 출력: 25

```

위 코드에서 **`PersonEntity`**는 엔티티, **`PersonDTO`**는 데이터 전송 객체입니다. **`ModelMapper`**를 사용하면 **`map()`** 메서드를 호출하여 두 객체 간의 필드를 자동으로 매핑할 수 있습니다.
