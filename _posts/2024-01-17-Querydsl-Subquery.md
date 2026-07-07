---
title: "[Querydsl] 서브쿼리(SubQuery)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, subquery]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **서브쿼리**를 정리한다.

> **서브쿼리란?** 하나의 SQL 문 안에서 또 다른 SQL 문을 실행하는 것. 복잡한 조건을 간단하게 처리할 때 쓴다.

> 💡 서브쿼리에는 **별칭이 다른 Q-Type**이 필요하다. (`QMember subMember = new QMember("subMember")`)

---

## 1. 단일 결과 서브쿼리

서브쿼리 결과가 **하나**일 때 (예: 최대 나이인 회원).

```java
QMember subMember = new QMember("subMember");

List<Member> members = queryFactory
    .selectFrom(member)
    .where(member.age.eq(                                    // 나이가 = 서브쿼리 결과
        JPAExpressions.select(subMember.age.max()).from(subMember)
    ))
    .fetch();
```

## 2. 다중 결과 서브쿼리

서브쿼리가 여러 값을 반환할 때 (예: 평균 이상 주문).

```java
QOrder subOrder = new QOrder("subOrder");

List<Order> orders = queryFactory
    .selectFrom(order)
    .where(order.orderAmount.gt(                             // > 평균
        JPAExpressions.select(subOrder.orderAmount.avg()).from(subOrder)
    ))
    .fetch();
```

## 3. exists 서브쿼리

**존재 여부**를 확인할 때.

```java
QMember subMember = new QMember("subMember");

List<Member> members = queryFactory
    .selectFrom(member)
    .where(JPAExpressions.selectFrom(subMember)
        .where(subMember.team.eq(member.team))
        .exists())   // 같은 팀 회원이 존재하면
    .fetch();
```

| 유형 | 용도 |
|------|------|
| 단일 결과 | `eq()`로 값 비교 (max 등) |
| 다중 결과 | `gt()`, `in()` 등 |
| exists | 존재 여부 (`exists`/`notExists`) |

---

## 📝 정리

```
QueryDSL 서브쿼리
├─ 준비   별칭 다른 Q-Type (new QMember("sub"))
├─ 단일   where(...eq(select max...))
├─ 다중   where(...gt(select avg...))
└─ exists 존재 여부 확인
```

| 개념 | 한 줄 정의 |
|------|------|
| **서브쿼리** | 쿼리 안의 쿼리 |
| **JPAExpressions** | 서브쿼리 작성 도구 |
| **exists** | 존재 여부 검사 |

서브쿼리는 복잡한 조건을 하나의 쿼리로 표현하게 해준다. 별칭이 다른 Q-Type이 필요하다는 점, 결과가 많으면 성능에 유의해야 한다는 점을 기억하자.
