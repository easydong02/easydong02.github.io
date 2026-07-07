---
title: "[Querydsl] 집계함수(Aggregate Function)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, aggregate, groupby]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 **집계 함수**를 정리한다.

> **집계 함수란?** 여러 행의 데이터를 하나로 합치거나, 조건에 따라 계산하는 함수. 데이터 분석·통계에 필수다.

---

## 1. count

```java
long productCount = queryFactory
    .selectFrom(product)
    .where(product.price.gt(1000))
    .fetchCount();   // 조건에 맞는 개수
```

---

## 2. sum · avg · min · max

숫자형 데이터의 합계·평균·최소·최대를 계산한다.

```java
BigDecimal totalAmount = queryFactory
    .select(product.amount.sum())   // 합계
    .from(product)
    .fetchOne();
```

| 함수 | 역할 |
|:---:|------|
| `.sum()` | 합계 |
| `.avg()` | 평균 |
| `.min()` / `.max()` | 최소 / 최대 |
| `.count()` / `fetchCount()` | 개수 |

---

## 3. groupBy와 함께

특정 기준으로 그룹핑한 뒤 집계한다. 여러 컬럼을 선택하면 **Tuple**로 받는다.

```java
List<Tuple> result = queryFactory
    .select(product.category, product.price.avg())   // 카테고리, 평균 가격
    .from(product)
    .groupBy(product.category)                        // 카테고리별
    .fetch();
```

> 💡 여러 값을 select하면 엔티티가 아니라 **`Tuple`**로 반환된다. `tuple.get(product.category)`처럼 꺼낸다.

---

## 📌 주의사항과 팁

> ⚠️ 집계는 DB에서 계산되므로, 데이터가 많으면 **성능에 영향**을 준다. 특히 `groupBy`는 그룹별로 무엇을 가져올지 명확히 정의해야 한다.

---

## 📝 정리

```
QueryDSL 집계
├─ count   fetchCount() / .count()
├─ 통계    sum/avg/min/max
├─ 그룹    groupBy(컬럼) → Tuple로 반환
└─ 주의    데이터 많으면 성능 고려
```

| 개념 | 한 줄 정의 |
|------|------|
| **집계 함수** | 여러 행을 하나로 계산 |
| **groupBy** | 기준별 그룹핑 후 집계 |
| **Tuple** | 여러 값 조회 결과 |

집계 함수는 SQL의 `SUM`/`AVG`/`GROUP BY`를 타입 안전하게 표현한 것이다. 여러 값을 select하면 `Tuple`로 받는다는 점, groupBy는 성능에 유의해야 한다는 점을 기억하자.
