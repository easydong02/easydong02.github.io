---
title: "[Querydsl] 페이징(paging)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, paging]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL로 **페이징**하는 방법을 정리한다.

> **페이징이란?** 데이터를 일정량씩 나눠 가져오는 것. 전체를 한 번에 로딩하지 않고 **필요한 만큼만** 가져와 성능을 최적화한다.

---

## 1. offset과 limit

```java
List<Product> products = queryFactory
    .selectFrom(product)
    .orderBy(product.price.asc())
    .offset(10)   // 10번째부터 시작
    .limit(5)     // 5개씩
    .fetch();
```

| 메소드 | 역할 |
|:---:|------|
| `offset(n)` | n번째부터 시작 (건너뛰기) |
| `limit(m)` | m개씩 가져오기 |

> 💡 페이징 시 **`orderBy`로 정렬을 명시**해야 데이터 순서가 일관된다.

---

## 2. fetchResults — 전체 개수까지

```java
QueryResults<Product> results = queryFactory
    .selectFrom(product)
    .orderBy(product.price.asc())
    .offset(10)
    .limit(5)
    .fetchResults();

List<Product> list = results.getResults();   // 현재 페이지 데이터
long total = results.getTotal();             // 전체 개수
```

> 💡 `fetchResults()`는 **한 번의 호출로 데이터와 전체 개수를 함께** 가져와, 전체 페이지 수 계산에 유용하다. (count 쿼리가 추가로 나감)

---

## 📌 주의사항과 팁

| 항목 | 내용 |
|------|------|
| **성능** | offset이 큰 경우 뒤로 갈수록 느려질 수 있음 |
| **fetchResults** | 데이터 + 전체 개수 → 페이지 수 계산에 유용 |
| **정렬 명시** | 페이징에는 정렬이 필수 |

---

## 📝 정리

```
QueryDSL 페이징
├─ offset(n)/limit(m)  건너뛰기 + 개수
├─ fetchResults()      데이터 + 전체 개수
└─ 필수                orderBy로 정렬 명시
```

| 개념 | 한 줄 정의 |
|------|------|
| **offset/limit** | 시작 위치 / 개수 |
| **fetchResults** | 결과 + 전체 개수 |

페이징은 `offset`·`limit`으로 간단히 구현한다. 전체 페이지 수가 필요하면 `fetchResults()`를 쓰고, 순서 일관성을 위해 항상 `orderBy`를 함께 지정하자.
