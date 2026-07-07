---
title: "[Querydsl] Fetch"
date: 2024-01-15 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, qtype]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL에서 쿼리 결과를 가져오는 **`fetch` 계열 메소드**들을 정리한다. 상황에 맞는 fetch를 골라 쓰는 것이 중요하다.

---

## 1. fetch 메소드의 종류

| 메소드 | 반환 | 특징 |
|------|------|------|
| `fetch()` | `List<T>` | 기본, 여러 결과 |
| `fetchOne()` | `T` | 단일 결과 (없거나 여럿이면 **예외**) |
| `fetchFirst()` | `T` | 첫 결과 (없으면 **null**) |
| `fetchResults()` | `QueryResults<T>` | 결과 + **전체 개수** (페이징) |

```java
// fetch() — 리스트
List<Product> products = queryFactory
    .selectFrom(product).where(product.price.gt(1000)).fetch();

// fetchOne() — 단일 (0개/2개↑ 시 예외)
Product one = queryFactory
    .selectFrom(product).where(product.id.eq(1L)).fetchOne();

// fetchFirst() — 첫 결과 (없으면 null)
Product first = queryFactory
    .selectFrom(product).where(product.category.eq("Electronics")).fetchFirst();

// fetchResults() — 결과 + 전체 개수
QueryResults<Product> results = queryFactory
    .selectFrom(product).where(product.price.between(500, 1000)).fetchResults();
List<Product> list = results.getResults();
long total = results.getTotal();
```

---

## 📌 사용법과 주의사항

| 상황 | 선택 |
|------|------|
| 여러 결과 | `fetch()` |
| 반드시 하나 | `fetchOne()` (예외 주의) |
| 있을 수도 없을 수도 | `fetchFirst()` |
| 페이징 (전체 개수 필요) | `fetchResults()` |

> ⚠️ `fetch`는 데이터를 메모리로 로딩하므로 **대량 데이터는 성능을 고려**해야 한다. 연관 엔티티를 함께 로딩하려면 `join`(fetch join)을 활용한다.

---

## 📝 정리

```
QueryDSL fetch
├─ fetch()         List 반환
├─ fetchOne()      단일 (없거나 여럿 → 예외)
├─ fetchFirst()    첫 결과 (없으면 null)
└─ fetchResults()  결과 + 전체 개수 (페이징)
```

| 메소드 | 한 줄 정의 |
|------|------|
| **fetch** | 여러 결과 리스트 |
| **fetchOne** | 정확히 하나 |
| **fetchFirst** | 첫 결과(없으면 null) |
| **fetchResults** | 결과+개수 |

fetch 메소드는 "결과가 몇 개인가, 개수가 필요한가"에 따라 골라 쓴다. 특히 `fetchOne()`은 결과가 0개나 2개 이상이면 예외가 나므로, 확실할 때만 사용하자.
