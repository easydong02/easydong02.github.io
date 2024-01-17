---
title: "[Querydsl] 케이스(Case)"
date: 2024-01-17 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, case]
render_with_liquid: false
future: true
---

# **QueryDSL - Case: 데이터에 따라 다르게 처리하기**

안녕하세요! 오늘은 QueryDSL에서 제공하는 **`case`** 표현식에 대해 알아보겠습니다. **`case`**는 데이터에 따라 다르게 처리해야 할 때 사용되며, 간단한 설명과 함께 다양한 **`case`** 표현식의 사용 예시 코드를 살펴봅시다.

## ✅**`case` 표현식이란?**

---

**`case`** 표현식은 SQL에서 조건에 따라 값을 다르게 반환하는 구문을 의미합니다. QueryDSL에서도 이와 유사한 방식으로 **`case`**를 사용하여 데이터에 따라 다르게 처리할 수 있습니다.

## ✅**`case` 사용하기**

---

### **간단한 `case` 표현식**

```java
javaCopy code
List<String> results = new JPAQueryFactory(entityManager)
    .select(member.age
        .when(10).then("10대")
        .when(20).then("20대")
        .otherwise("기타"))
    .from(member)
    .fetch();

```

위 코드에서는 **`age`** 컬럼의 값이 10이면 "10대", 20이면 "20대", 그 외에는 "기타"를 반환합니다.

### **복잡한 `case` 표현식**

```java
javaCopy code
List<String> results = new JPAQueryFactory(entityManager)
    .select(
        new CaseBuilder()
            .when(member.age.lt(20)).then("10대 미만")
            .when(member.age.between(20, 29)).then("20대")
            .when(member.age.between(30, 39)).then("30대")
            .otherwise("기타")
    )
    .from(member)
    .fetch();

```

위 코드에서는 **`age`** 컬럼의 값에 따라 다양한 조건을 사용하여 반환값을 설정하고 있습니다.

## ✅**`case`를 활용한 동적 쿼리**

---

**`case`**를 활용하면 동적인 쿼리를 쉽게 작성할 수 있습니다.

### **예시 코드**

```java
javaCopy code
BooleanBuilder builder = new BooleanBuilder();

// 조건에 따라 동적으로 쿼리 작성
if (condition1) {
    builder.and(member.age.gt(20));
}

if (condition2) {
    builder.and(member.name.like("홍길동%"));
}

List<Member> results = new JPAQueryFactory(entityManager)
    .selectFrom(member)
    .where(builder)
    .fetch();

```

위 코드에서는 **`case`**를 사용하여 동적인 조건을 빌더를 활용해 쉽게 작성할 수 있습니다.

## 📌**주의사항과 팁**

---

- **`case`** 표현식은 복잡한 조건을 간결하게 표현할 수 있으나, 지나치게 복잡하게 사용할 경우 가독성이 떨어질 수 있습니다.
- 동적 쿼리를 작성할 때 **`case`**를 활용하면 가독성이 높아질 수 있습니다.
- 복잡한 조건이나 동적인 쿼리에 대해서는 필요에 따라 **`case`** 표현식을 활용하여 코드를 간결하게 작성할 수 있습니다.