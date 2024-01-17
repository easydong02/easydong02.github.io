---
title: "[Querydsl] 조인(join)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, join, fetch]
render_with_liquid: false
future: true
---

# **QueryDSL - Join: 다양한 Join 방식으로 데이터 연결하기**

안녕하세요! 오늘은 QueryDSL에서 제공하는 **`join`**에 대해 알아보겠습니다. 데이터베이스에서 여러 테이블 간에 관계가 형성되어 있을 때, 이를 활용하여 효과적으로 데이터를 가져오는 방법을 살펴보겠습니다. 다양한 Join 방식과 함께 예시 코드를 통해 실제 사용법을 익혀봅시다.

## ✅**Inner Join**

---

Inner Join은 두 테이블 간에 일치하는 데이터만을 가져옵니다.

### **예시 코드**

```java

List<Order> orders = new JPAQueryFactory(entityManager)
    .selectFrom(order)
    .innerJoin(order.customer, customer)
    .fetch();

```

위 코드에서는 **`order`** 테이블과 **`customer`** 테이블을 Inner Join하여 관련된 주문 정보를 가져옵니다.

## ✅**Left Join**

---

Left Join은 왼쪽 테이블의 모든 데이터를 가져오고, 오른쪽 테이블과 일치하는 데이터가 있으면 함께 가져옵니다.

### **예시 코드**

```java

List<Order> orders = new JPAQueryFactory(entityManager)
    .selectFrom(order)
    .leftJoin(order.customer, customer)
    .fetch();

```

위 코드에서는 **`order`** 테이블을 기준으로 Left Join하여 모든 주문 정보를 가져오고, 해당 주문과 연결된 고객 정보가 있다면 함께 가져옵니다.

## ✅**Fetch Join**

---

Fetch Join은 연관된 엔티티를 함께 로딩하여 N+1 문제를 해결하는 방식입니다.

### **예시 코드**

```java

List<Order> orders = new JPAQueryFactory(entityManager)
    .selectFrom(order)
    .innerJoin(order.customer, customer).fetchJoin()
    .fetch();

```

위 코드에서는 **`order`** 테이블과 **`customer`** 테이블을 Inner Join하면서 Fetch Join을 적용하여 관련된 고객 정보를 함께 로딩합니다.

## ✅**On 절 활용**

---

**`on`** 절을 사용하여 Join 조건을 추가할 수 있습니다.

### **예시 코드**

```java

List<Order> orders = new JPAQueryFactory(entityManager)
    .selectFrom(order)
    .leftJoin(order.customer, customer)
    .on(customer.grade.eq("VIP"))
    .fetch();

```

위 코드에서는 **`customer`** 테이블과 **`VIP`** 등급을 가진 주문 정보만을 Left Join하여 가져옵니다.

## 📌**주의사항과 팁**

---

- Join을 사용할 때는 데이터베이스의 인덱스 등을 고려하여 성능을 최적화해야 합니다.
- Fetch Join은 N+1 문제를 해결하지만, 데이터 양이 많을 경우 부하를 일으킬 수 있으므로 주의가 필요합니다.
- Join 조건을 명확하게 설정하여 의도하지 않은 결과가 나오지 않도록 주의해야 합니다.