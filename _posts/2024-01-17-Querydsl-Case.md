---
title: "[Querydsl] 케이스(Case)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, case]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **`case` 표현식**을 정리한다.

> **`case` 표현식이란?** SQL에서 **조건에 따라 값을 다르게 반환**하는 구문. QueryDSL에서도 데이터에 따라 다르게 처리할 때 사용한다.

---

## 1. 간단한 case

```java
List<String> results = queryFactory
    .select(member.age
        .when(10).then("10대")
        .when(20).then("20대")
        .otherwise("기타"))   // 그 외
    .from(member)
    .fetch();
```

값이 정확히 일치할 때 쓰는 방식이다.

## 2. 복잡한 case (CaseBuilder)

**범위 조건**은 `CaseBuilder`로 처리한다.

```java
List<String> results = queryFactory
    .select(new CaseBuilder()
        .when(member.age.lt(20)).then("10대 미만")
        .when(member.age.between(20, 29)).then("20대")
        .when(member.age.between(30, 39)).then("30대")
        .otherwise("기타"))
    .from(member)
    .fetch();
```

| 방식 | 용도 |
|------|------|
| `.when(값).then()` | 정확 일치 |
| `new CaseBuilder().when(조건)` | 범위·복잡 조건 |

---

## 3. 동적 쿼리 (BooleanBuilder)

조건에 따라 쿼리를 **동적으로 조립**한다.

```java
BooleanBuilder builder = new BooleanBuilder();
if (condition1) builder.and(member.age.gt(20));
if (condition2) builder.and(member.name.like("홍길동%"));

List<Member> results = queryFactory
    .selectFrom(member)
    .where(builder)   // 동적 조건
    .fetch();
```

---

## 📝 정리

```
QueryDSL case
├─ 간단   .when(값).then().otherwise()
├─ 복잡   new CaseBuilder().when(조건)
└─ 동적   BooleanBuilder로 조건 조립
```

| 개념 | 한 줄 정의 |
|------|------|
| **case** | 조건별 값 반환 |
| **CaseBuilder** | 범위·복잡 조건 처리 |
| **BooleanBuilder** | 동적 조건 조립 |

`case`는 조건에 따라 값을 바꿀 때, `BooleanBuilder`는 동적 쿼리를 만들 때 쓴다. 다만 지나치게 복잡하면 가독성이 떨어지니 적절히 활용하자.
