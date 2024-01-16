---
title: "[Querydsl] 정렬(Sort)"
date: 2024-01-16 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, sort, orderby]
render_with_liquid: false
future: true
---

# **QueryDSL - Sort: 데이터 정렬을 손쉽게!**

안녕하세요! 오늘은 QueryDSL에서 데이터를 정렬하는데 사용되는 **`sort`**에 대해 알아보겠습니다. 데이터를 특정 기준에 따라 정렬하는 것은 많은 상황에서 필요한데, QueryDSL은 이를 간단하게 처리할 수 있는 다양한 기능을 제공합니다. 함께 예시 코드와 함께 살펴보도록 하겠습니다.

## ✅**Sort란?**

---

정렬은 데이터를 특정 기준에 따라 순서대로 나열하는 것을 말합니다. QueryDSL에서는 **`orderBy`**와 **`sort`**를 통해 데이터를 정렬할 수 있습니다. **`orderBy`**는 JPQL과 유사한 방식으로 사용되고, **`sort`**는 Java 8부터 도입된 **`Comparator`**를 활용하여 더 유연하게 정렬할 수 있습니다.

## ✅**`orderBy`를 사용한 정렬**

---

**`orderBy`**를 사용하면 간단하게 엔티티의 필드에 따라 정렬할 수 있습니다.

### **예시 코드**

```java
javaCopy code
List<Product> products = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .orderBy(product.price.desc(), product.name.asc())
    .fetch();

```

위 코드에서 **`orderBy(product.price.desc(), product.name.asc())`**는 **`price`**를 내림차순으로 정렬하고, 그 다음에 **`name`**을 오름차순으로 정렬합니다.

## ✅**`sort`를 사용한 정렬**

---

**`sort`**를 사용하면 Java 8의 **`Comparator`**를 이용하여 더 유연하게 정렬할 수 있습니다.

### **예시 코드**

```java
javaCopy code
List<Product> products = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .fetch()
    .stream()
    .sorted(Comparator.comparing(Product::getPrice).reversed())
    .collect(Collectors.toList());

```

위 코드에서 **`Comparator.comparing(Product::getPrice).reversed()`**는 **`price`**를 내림차순으로 정렬합니다.

## ✅**다중 정렬**

---

여러 필드에 대해 다중 정렬도 간단하게 처리할 수 있습니다.

### **예시 코드**

```java
javaCopy code
List<Product> products = new JPAQueryFactory(entityManager)
    .selectFrom(product)
    .orderBy(product.category.asc(), product.price.desc())
    .fetch();

```

위 코드에서는 **`category`**를 오름차순으로 정렬하고, 그 다음에 **`price`**를 내림차순으로 정렬합니다.

## 📌**주의사항과 팁**

- **`orderBy`**나 **`sort`**는 특정 필드에 대한 정렬 외에도 **`nullsFirst()`**, **`nullsLast()`** 등을 통해 **`null`** 값을 어떻게 처리할지 설정할 수 있습니다.
- 다양한 정렬 조건을 유연하게 활용하여 필요에 맞게 데이터를 정렬하세요.
- 정렬은 데이터베이스에서 가져온 이후에 이루어지므로, 데이터베이스에서 정렬하는 것보다 자원 소모가 높을 수 있습니다.