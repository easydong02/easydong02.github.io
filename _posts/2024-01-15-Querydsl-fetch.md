---
title: "[Querydsl] Fetch"
date: 2024-01-15 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, qtype]
render_with_liquid: false
future: true
---

# **QueryDSL - Fetch의 종류와 사용법: 데이터 효율적으로 가져오기**

안녕하세요! 오늘은 QueryDSL에서 사용되는 **`fetch`** 메서드에 대해 자세히 알아보겠습니다. **`fetch`** 메서드는 쿼리 결과를 가져오는데 있어 여러 가지 옵션을 제공합니다. 어떤 상황에서 어떤 **`fetch`** 메서드를 사용해야 하는지 알아보면서 예시 코드와 함께 살펴보겠습니다.

## ✅**`fetch` 메서드의 종류**

---

QueryDSL에서는 다양한 **`fetch`** 메서드를 제공하고 있습니다. 주로 사용되는 **`fetch`** 메서드에 대해 알아보겠습니다.

### **1. `fetch()`**

가장 기본적인 형태의 **`fetch`** 메서드로, 쿼리 결과를 리스트로 반환합니다.

```java

List<Product> products = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .where(product.price.gt(1000))
    .fetch();

```

### **2. `fetchOne()`**

단일 결과를 반환하며, 결과가 없거나 여러 개인 경우 예외를 발생시킵니다.

```java

Product product = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .where(product.id.eq(1L))
    .fetchOne();

```

### **3. `fetchFirst()`**

첫 번째 결과를 반환하며, 결과가 없는 경우 **`null`**을 반환합니다.

```java

Product product = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .where(product.category.eq("Electronics"))
    .fetchFirst();

```

### **4. `fetchResults()`**

결과를 페이징하여 반환하며, 전체 결과 개수도 함께 가져옵니다.

```java

QueryResults<Product> results = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .where(product.price.between(500, 1000))
    .fetchResults();

List<Product> productList = results.getResults();
long total = results.getTotal();

```

## ✅**`fetch` 메서드 사용법과 주의사항**

---

### 📌**사용법**

- **`fetch`** 메서드는 기본적으로 JPQL 쿼리를 생성하고 실행합니다. 따라서 기존의 JPQL 쿼리와 비슷한 형태로 작성할 수 있습니다.
- **`fetchOne()`**과 **`fetchFirst()`**는 단일 결과를 반환하므로, 결과가 없거나 여러 개인 경우 주의가 필요합니다.
- **`fetchResults()`**는 페이징 처리에 유용하게 사용할 수 있습니다.

### 📌**주의사항**

- **`fetch`** 메서드는 데이터를 메모리로 전체 로딩하므로, 대량의 데이터를 처리할 때는 성능 고려가 필요합니다.
- **`fetch`** 메서드는 즉시 로딩을 수행하므로, 연관된 엔티티를 함께 로딩하려면 **`join`**과 같은 기능을 활용해야 합니다.
- 쿼리 성능을 최적화하려면 **`fetch`** 메서드의 다양한 옵션을 활용하여 필요한 데이터만을 효율적으로 가져오도록 노력해야 합니다.
