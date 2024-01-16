---
title: "[Querydsl] 집계함수(Aggregate Function)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, aggregate, groupby]
render_with_liquid: false
future: true
---

# **QueryDSL - 집계 함수: 데이터를 효과적으로 분석하자!**

안녕하세요! 오늘은 QueryDSL에서 제공하는 집계 함수에 대해 알아보겠습니다. 집계 함수는 데이터를 분석하고 통계 내는 데에 필수적인 기능 중 하나입니다. QueryDSL은 다양한 집계 함수를 제공하여 데이터를 효과적으로 다룰 수 있습니다. 함께 예시 코드와 함께 살펴보도록 하겠습니다.

## ✅**집계 함수란?**

---

집계 함수는 여러 행의 데이터를 하나로 합치거나 특정 조건에 따라 데이터를 계산하는 함수입니다. QueryDSL에서는 다양한 집계 함수를 지원하여 데이터를 효과적으로 처리할 수 있습니다.

## ✅**`count` 함수 사용하기**

---

**`count`** 함수는 특정 조건에 맞는 데이터의 개수를 세는 데에 사용됩니다.

### **예시 코드**

```java
javaCopy code
long productCount = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .where(product.price.gt(1000))
    .fetchCount();

```

위 코드에서 **`product.price.gt(1000)`**은 가격이 1000 이상인 제품의 개수를 세는 조건이며, **`fetchCount()`**를 통해 개수를 가져옵니다.

## ✅**`sum`, `avg`, `min`, `max` 함수 사용하기**

---

수량, 가격 등의 숫자형 데이터에 대한 합계, 평균, 최솟값, 최댓값을 계산할 수 있습니다.

### **예시 코드**

```java
javaCopy code
BigDecimal totalAmount = new JPAQueryFactory(entityManager)
    .select(product.amount.sum())
    .from(product)
    .fetchOne();

```

위 코드에서 **`product.amount.sum()`**은 수량(amount)의 합계를 계산합니다.

## ✅**`groupBy`와 함께 사용하기**

---

**`groupBy`** 함수를 사용하면 특정 기준에 따라 데이터를 그룹핑하고 집계 함수를 적용할 수 있습니다.

### **예시 코드**

```java
javaCopy code
List<Tuple> result = new JPAQueryFactory(entityManager)
    .select(product.category, product.price.avg())
    .from(product)
    .groupBy(product.category)
    .fetch();

```

위 코드에서는 카테고리별로 가격의 평균을 계산합니다.

## 📌**주의사항과 팁**

---

- 집계 함수는 데이터베이스에서 계산되므로, 데이터 양이 많을 경우 성능에 영향을 줄 수 있습니다. 특히 **`groupBy`**를 사용할 때는 주의가 필요합니다.
- 집계 함수를 사용할 때는 필요한 데이터만을 선택하여 최적화된 쿼리를 작성하는 것이 중요합니다.
- **`groupBy`** 함수를 사용할 때는 그룹별로 어떤 데이터를 가져올지 명확하게 정의해야 합니다.