---
title: "[Querydsl] 서브쿼리(SubQuery)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, subquery]
render_with_liquid: false
future: true
---

# **QueryDSL - 서브쿼리: 데이터베이스를 더 효과적으로 활용하기**

안녕하세요! 오늘은 QueryDSL에서 제공하는 서브쿼리에 대해 알아보겠습니다. 서브쿼리는 주 쿼리 안에서 또 다른 쿼리를 실행하는 기술로, 데이터베이스를 더 효과적으로 활용하는 데에 도움을 줍니다. 간단한 설명과 함께 다양한 서브쿼리의 활용 예시 코드를 살펴봅시다.

## ✅**서브쿼리란?**

---

서브쿼리는 하나의 SQL 문 안에서 또 다른 SQL 문을 실행하는 것을 의미합니다. QueryDSL에서는 서브쿼리를 통해 복잡한 조건을 간단하게 처리하고자 할 때 사용됩니다.

## ✅**단일 결과 서브쿼리**

---

서브쿼리가 반환하는 결과가 하나인 경우에 사용됩니다.

### **예시 코드**

```java
javaCopy code
QMember subMember = new QMember("subMember");

List<Member> members = new JPAQueryFactory(entityManager)
    .selectFrom(member)
    .where(member.age.eq(
        new JPASubQuery().select(subMember.age.max()).from(subMember)
    ))
    .fetch();

```

위 코드에서는 **`Member`** 테이블에서 **`age`**가 최대인 회원을 찾아와서 해당 나이와 동일한 회원을 찾아옵니다.

## ✅**다중 결과 서브쿼리**

---

서브쿼리가 여러 결과를 반환하는 경우에 사용됩니다.

### **예시 코드**

```java
javaCopy code
QOrder subOrder = new QOrder("subOrder");

List<Order> orders = new JPAQueryFactory(entityManager)
    .selectFrom(order)
    .where(order.orderAmount.gt(
        new JPASubQuery().select(subOrder.orderAmount.avg()).from(subOrder)
    ))
    .fetch();

```

위 코드에서는 **`Order`** 테이블에서 주문 금액이 평균 이상인 주문을 찾아옵니다.

## ✅**`exists` 서브쿼리**

---

**`exists`** 키워드를 사용하여 서브쿼리의 결과의 존재 여부를 확인할 수 있습니다.

### **예시 코드**

```java
javaCopy code
QMember subMember = new QMember("subMember");

List<Member> members = new JPAQueryFactory(entityManager)
    .selectFrom(member)
    .where(new JPASubQuery().from(subMember)
        .where(subMember.team.eq(member.team))
        .exists())
    .fetch();

```

위 코드에서는 **`Member`** 테이블에서 동일한 팀에 속한 회원이 있는 경우에만 결과를 가져옵니다.

## 📌**주의사항과 팁**

---

- 서브쿼리를 사용할 때는 결과가 어떤 형태로 나올지 고려하여 주어진 상황에 맞게 적절한 방식을 선택해야 합니다.
- 서브쿼리의 결과가 너무 많아질 경우 성능에 영향을 줄 수 있으므로 주의가 필요합니다.
- 필요에 따라 **`exists`**, **`notExists`** 등을 활용하여 존재 여부를 확인할 수 있습니다.