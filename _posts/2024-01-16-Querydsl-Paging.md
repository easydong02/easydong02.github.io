---
title: "[Querydsl] 페이징(paging)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, paging]
render_with_liquid: false
future: true
---

# **QueryDSL - Paging: 데이터를 나눠서 효율적으로 가져오기**

안녕하세요! 오늘은 QueryDSL에서 데이터를 페이징하는 방법에 대해 알아보겠습니다. 페이징은 대량의 데이터를 조금씩 나눠서 가져오는 기술로, QueryDSL은 이를 간단하게 처리할 수 있는 다양한 기능을 제공합니다. 함께 예시 코드와 함께 살펴보겠습니다.

## ✅**페이징이란?**

---

페이징은 데이터를 일정량씩 나눠서 가져오는 것을 말합니다. 이는 대량의 데이터를 처리할 때 전체 데이터를 한 번에 로딩하지 않고, 필요한 만큼 가져와서 성능을 최적화하는 데 사용됩니다.

## ✅**`offset`과 `limit`을 활용한 페이징**

---

QueryDSL에서는 **`offset`**과 **`limit`**을 사용하여 페이징을 할 수 있습니다.

### **예시 코드**

```java
javaCopy code
List<Product> products = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .orderBy(product.price.asc())
    .offset(10) // 10번째부터 시작
    .limit(5)  // 5개씩 가져오기
    .fetch();

```

위 코드에서 **`offset(10)`**은 10번째부터 시작하여 데이터를 가져오고, **`limit(5)`**는 5개씩 가져온다는 의미입니다.

## ✅**`fetchResults`를 활용한 페이징**

---

**`fetchResults`** 메서드는 결과를 페이징하여 반환하며, 전체 결과 개수도 함께 가져옵니다.

### **예시 코드**

```java
javaCopy code
QueryResults<Product> results = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .orderBy(product.price.asc())
    .offset(10)
    .limit(5)
    .fetchResults();

List<Product> productList = results.getResults();
long total = results.getTotal();

```

위 코드에서 **`fetchResults`**를 사용하면 **`List`**와 함께 전체 결과 개수를 알 수 있어 페이징 처리에 유용합니다.

## 📌**주의사항과 팁**

---

- 페이징은 대체로 데이터베이스에서 일어나는 작업이므로, 성능 상의 이슈에 주의해야 합니다.
- **`offset`**과 **`limit`**을 사용할 때, 데이터베이스에서는 전체 데이터를 가져와서 자바에서 **`offset`**과 **`limit`**을 적용하는 방식으로 동작할 수 있습니다. 따라서 대량의 데이터를 다룰 때는 성능 고려가 필요합니다.
- **`fetchResults`**를 사용하면 한 번의 쿼리로 결과와 전체 데이터 개수를 가져올 수 있어, 전체 페이지 수 등을 계산하는 데 유용합니다.