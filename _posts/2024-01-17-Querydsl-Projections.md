---
title: "[Querydsl] 프로젝션(Projections)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, projecdtions]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **Projections**를 정리한다.

> **Projections란?** 쿼리 결과를 **원하는 형태(단일 값·DTO·Tuple)로 가져오는** 기능. 엔티티 전체가 아니라 필요한 필드만 뽑아올 수 있다.

---

## 1. 기본 Projection (단일 값)

```java
List<String> memberNames = queryFactory
    .select(member.name)   // name만
    .from(member)
    .fetch();
```

## 2. DTO Projection — 3가지 방식

여러 필드를 DTO로 받는 방법은 세 가지다.

| 방식 | 매핑 기준 |
|------|------|
| **`Projections.bean`** | **Setter** |
| **`Projections.fields`** | **필드**(직접) |
| **`Projections.constructor`** | **생성자** |

```java
// bean (Setter 기반)
Projections.bean(MemberDto.class, member.name, member.age)

// constructor (생성자 기반)
Projections.constructor(MemberDto.class, member.name, member.age)

// fields + 별칭(as)
Projections.fields(MemberProjection.class,
    member.name.as("username"),   // 필드명이 다르면 as로 매핑
    member.age)
```

```java
List<MemberDto> dtos = queryFactory
    .select(Projections.constructor(MemberDto.class, member.name, member.age))
    .from(member)
    .fetch();
```

> 💡 필드명이 DTO와 다르면 **`.as("별칭")`**으로 맞춘다.

## 3. Tuple Projection

여러 값을 **Tuple**로 받는다.

```java
List<Tuple> tuples = queryFactory
    .select(member.name, member.age)   // 여러 값 → Tuple
    .from(member)
    .fetch();
// tuple.get(member.name), tuple.get(member.age)로 접근
```

---

## 📌 주의사항과 팁

> 💡 **DTO Projection**을 쓰면 엔티티-DTO 간 의존성을 낮추고 **필요한 필드만** 가져와, 성능·메모리 측면에서 효과적이다.

---

## 📝 정리

```
QueryDSL Projections
├─ 단일 값   .select(member.name)
├─ DTO      bean(Setter) / fields(필드) / constructor(생성자)
├─ 별칭      .as("username")
└─ Tuple    여러 값 → Tuple
```

| 개념 | 한 줄 정의 |
|------|------|
| **Projections** | 결과를 원하는 형태로 |
| **bean/fields/constructor** | Setter/필드/생성자 매핑 |
| **Tuple** | 여러 값 담는 컨테이너 |

Projections는 "필요한 것만 가져오기"의 도구다. 특히 **DTO Projection**은 엔티티 노출을 막고 성능도 좋아, 조회 API에서 널리 쓰인다. 매핑 방식(bean/fields/constructor)의 차이를 기억하자.
