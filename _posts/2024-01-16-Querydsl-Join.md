---
title: "[Querydsl] 조인(join)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, join, fetch]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **조인(Join)**을 정리한다. 여러 테이블 간 관계를 활용해 데이터를 연결하는 다양한 방식을 살펴본다.

| 종류 | 설명 |
|------|------|
| **Inner Join** | 양쪽에 **일치하는 것만** |
| **Left Join** | 왼쪽 전부 + 일치하는 오른쪽 |
| **Fetch Join** | 연관 엔티티를 **함께 로딩**(N+1 해결) |

---

## 1. Inner Join

```java
List<Order> orders = queryFactory
    .selectFrom(order)
    .innerJoin(order.customer, customer)   // 일치하는 것만
    .fetch();
```

## 2. Left Join

```java
List<Order> orders = queryFactory
    .selectFrom(order)
    .leftJoin(order.customer, customer)    // order 전부 + 있으면 customer
    .fetch();
```

## 3. Fetch Join — N+1 해결

```java
List<Order> orders = queryFactory
    .selectFrom(order)
    .innerJoin(order.customer, customer).fetchJoin()   // 함께 로딩
    .fetch();
```

> 💡 **Fetch Join**은 연관 엔티티(customer)를 **한 번의 쿼리로 함께** 가져와 **N+1 문제**를 해결한다. (일반 join은 데이터를 연결만 하고, 연관 엔티티는 지연 로딩됨)

## 4. On 절 — 조인 조건 추가

```java
List<Order> orders = queryFactory
    .selectFrom(order)
    .leftJoin(order.customer, customer)
    .on(customer.grade.eq("VIP"))          // 조인 조건 추가
    .fetch();
```

> 💡 `on`으로 조인 시점의 추가 조건을 건다. (VIP 등급 고객만 연결)

---

## 📌 주의사항과 팁

> ⚠️ **Fetch Join은 N+1을 해결하지만, 데이터가 많으면 부하**를 일으킬 수 있다. 조인 조건을 명확히 해 의도치 않은 결과를 막자. 인덱스도 고려하면 성능이 향상된다.

---

## 📝 정리

```
QueryDSL 조인
├─ innerJoin   일치하는 것만
├─ leftJoin    왼쪽 전부 + 일치 오른쪽
├─ fetchJoin() 연관 엔티티 함께 로딩 (N+1 해결)
└─ on()        조인 조건 추가
```

| 개념 | 한 줄 정의 |
|------|------|
| **Inner/Left Join** | 교집합 / 왼쪽 기준 |
| **Fetch Join** | 연관 엔티티 함께 로딩 |
| **on** | 조인 시 추가 조건 |

조인의 핵심은 **Fetch Join으로 N+1을 해결**한다는 것이다. 다만 데이터가 많으면 부하가 될 수 있으니, 조인 조건과 인덱스를 함께 고려해 최적화하자.
