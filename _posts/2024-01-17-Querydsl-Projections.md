---
title: "[Querydsl] 프로젝션(Projections)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, projecdtions]
render_with_liquid: false
future: true
---

# **QueryDSL - Projections: 결과를 다양하게 펼쳐보자!**

안녕하세요! 오늘은 QueryDSL에서 제공하는 **`Projections`**에 대해 알아보겠습니다. **`Projections`**는 쿼리의 결과를 다양한 형태로 가져올 수 있는 기능으로, 간단한 설명과 함께 다양한 **`Projections`** 종류와 예시 코드를 살펴봅시다.

## ✅**Projections이란?**

---

**`Projections`**는 쿼리의 결과를 특정 형태로 가져올 때 사용되는 기능입니다. QueryDSL에서는 다양한 종류의 **`Projections`**를 제공하여 원하는 형태로 결과를 펼칠 수 있습니다.

## ✅**기본적인 Projections 사용하기**

---

```java

List<String> memberNames = new JPAQueryFactory(entityManager)
    .select(member.name)
    .from(member)
    .fetch();

```

위 코드에서는 **`member`** 엔티티의 **`name`** 필드만을 선택하여 결과를 가져오고 있습니다. 이것이 기본적인 **`Projections`**의 사용 방법입니다.

## ✅**DTO Projections 사용하기**

---

```java

List<MemberDto> memberDtos = new JPAQueryFactory(entityManager)
    .select(Projections.bean(MemberDto.class,
        member.name,
        member.age))
    .from(member)
    .fetch();

```

위 코드에서는 **`MemberDto`** 클래스에 대한 DTO Projections을 사용하여 원하는 필드만을 선택하여 결과를 가져오고 있습니다. DTO Projections은 원하는 형태의 결과를 만들어주는 데에 유용합니다.

## ✅**생성자 Projections 사용하기**

---

```java

List<MemberDto> memberDtos = new JPAQueryFactory(entityManager)
    .select(Projections.constructor(MemberDto.class,
        member.name,
        member.age))
    .from(member)
    .fetch();

```

DTO Projections과 유사하게, 생성자 Projections을 사용하여 DTO 객체를 생성하고 필요한 필드만을 선택할 수 있습니다.

## ✅**Tuple Projections 사용하기**

---

```java

List<Tuple> tuples = new JPAQueryFactory(entityManager)
    .select(member.name, member.age)
    .from(member)
    .fetch();

```

Tuple Projections은 여러 필드를 튜플 형태로 가져올 수 있습니다. 이 때, 각 필드에 대한 별칭을 지정하지 않으면 튜플의 각 원소는 인덱스로 접근할 수 있습니다.

## ✅**Q-Type Projections 사용하기**

---

```java

List<MemberProjection> projections = new JPAQueryFactory(entityManager)
    .select(Projections.fields(MemberProjection.class,
        member.name.as("username"),
        member.age))
    .from(member)
    .fetch();

```

Q-Type Projections은 엔티티 대신 Q-Type을 사용하여 결과를 가져올 수 있습니다. 이를 통해 컴파일 시점에 타입 안정성을 보장할 수 있습니다.

## 📌**주의사항과 팁**

---

- **`Projections`**를 활용하면 쿼리 결과를 원하는 형태로 편리하게 가져올 수 있습니다.
- DTO Projections을 사용하여 엔티티와 DTO 사이의 의존성을 낮추고 필요한 필드만을 가져오는 것이 성능 및 메모리 사용 측면에서 효과적입니다.
- **`Projections`**를 적절히 사용하여 데이터를 효율적으로 처리하세요.