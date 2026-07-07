---
title: "[Querydsl] 정렬(Sort)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, sort, orderby]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **정렬**을 정리한다.

> QueryDSL에서 정렬은 두 가지 방식이 있다.
> - **`orderBy`**: DB 단에서 정렬 (JPQL 유사)
> - **`Comparator`(sort)**: 가져온 뒤 자바에서 정렬 (Java 8 스트림)

---

## 1. orderBy — DB 정렬

```java
List<Product> products = queryFactory
    .selectFrom(product)
    .orderBy(product.price.desc(), product.name.asc())   // 가격↓, 이름↑
    .fetch();
```

> 💡 여러 정렬 기준을 콤마로 나열하면 **순서대로 우선 적용**된다. (가격 내림차순 → 같으면 이름 오름차순)

## 2. Comparator — 자바 정렬

```java
List<Product> products = queryFactory
    .selectFrom(product).fetch()
    .stream()
    .sorted(Comparator.comparing(Product::getPrice).reversed())   // 가격↓
    .collect(Collectors.toList());
```

| 방식 | 정렬 위치 | 특징 |
|------|:---:|------|
| **orderBy** | **DB** | 효율적 (권장) |
| **Comparator** | 자바(메모리) | 유연하나 자원 소모↑ |

## 3. 다중 정렬

```java
.orderBy(product.category.asc(), product.price.desc())   // 카테고리↑, 가격↓
```

---

## 📌 주의사항과 팁

> 💡 `nullsFirst()`, `nullsLast()`로 **null 값의 정렬 위치**를 지정할 수 있다.

> ⚠️ `Comparator` 정렬은 **DB에서 가져온 이후** 이뤄지므로, DB 정렬(`orderBy`)보다 자원 소모가 크다. 가급적 `orderBy`를 쓰자.

---

## 📝 정리

```
QueryDSL 정렬
├─ orderBy      DB 정렬 (권장), 다중 기준 나열
├─ Comparator   자바 정렬 (유연, 자원↑)
└─ null 처리    nullsFirst() / nullsLast()
```

| 개념 | 한 줄 정의 |
|------|------|
| **orderBy** | DB에서 정렬 |
| **Comparator** | 자바에서 정렬 |
| **다중 정렬** | 여러 기준 순서대로 |

정렬은 **가급적 `orderBy`(DB)**를 쓰는 게 효율적이다. 자바 `Comparator`는 유연하지만 메모리에서 정렬하므로 자원 소모가 크다는 점을 기억하자.
