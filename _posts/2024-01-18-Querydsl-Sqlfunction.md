---
title: "[Querydsl] sql함수(Sql functions)"
date: 2024-01-18 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, sqlfunction]
render_with_liquid: false
future: true
---

# **QueryDSL - SQL Function: 데이터를 더 유연하게 다루자!**

안녕하세요! 오늘은 QueryDSL에서 제공하는 **`SQL Function`**에 대해 알아보겠습니다. **`SQL Function`**은 데이터를 가공하고 쿼리를 작성할 때 유용한 기능으로, 다양한 함수들을 활용하여 원하는 결과를 얻을 수 있습니다. 지금부터는 그 중 몇 가지를 예시 코드와 함께 살펴보겠습니다.

## ✅**SQL Function이란?**

---

**`SQL Function`**은 데이터베이스에서 제공하는 다양한 함수를 QueryDSL에서 활용할 수 있게 해주는 기능입니다. 이를 사용하여 쿼리를 더 유연하게 작성할 수 있습니다.

## ✅**문자열 함수 활용하기**

---

### **`concat` 함수**

```java
javaCopy code
List<String> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.concat(member.name, "(", member.age, ")"))
    .from(member)
    .fetch();

```

위 코드에서는 **`concat`** 함수를 사용하여 **`name`**과 **`age`**를 합친 문자열을 가져오고 있습니다. 결과로는 "이름(나이)" 형태의 문자열이 반환됩니다.

### **`lower` 함수**

```java
javaCopy code
List<String> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.lower(member.name))
    .from(member)
    .fetch();

```

**`lower`** 함수를 사용하여 **`name`**의 모든 문자를 소문자로 변환한 결과를 가져옵니다.

## ✅**날짜 함수 활용하기**

---

### **`currentDate` 함수**

```java
javaCopy code
List<Date> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.currentDate())
    .from(member)
    .fetch();

```

**`currentDate`** 함수를 사용하여 현재 날짜를 가져옵니다.

### **`addDays` 함수**

```java
javaCopy code
List<Date> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.addDays(member.joinDate, 7))
    .from(member)
    .fetch();

```

**`addDays`** 함수를 사용하여 **`joinDate`**에 7일을 더한 결과를 가져옵니다.

## ✅**수학 함수 활용하기**

---

### **`sqrt` 함수**

```java
javaCopy code
List<Double> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.sqrt(member.age.doubleValue()))
    .from(member)
    .fetch();

```

**`sqrt`** 함수를 사용하여 **`age`**의 제곱근을 가져옵니다.

### **`round` 함수**

```java
javaCopy code
List<Integer> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.round(member.height))
    .from(member)
    .fetch();

```

**`round`** 함수를 사용하여 **`height`**를 반올림한 결과를 가져옵니다.

## ✅**`coalesce` 함수 활용하기**

---

```java
javaCopy code
List<String> results = new JPAQueryFactory(entityManager)
    .select(SQLExpressions.coalesce(member.nickname, "닉네임 없음"))
    .from(member)
    .fetch();

```

**`coalesce`** 함수를 사용하여 **`nickname`**이 **`null`**일 경우 기본값인 "닉네임 없음"을 반환합니다.

## 📌**주의사항과 팁**

---

- SQL Function은 데이터베이스 종류에 따라 지원되는 함수가 다를 수 있으므로 확인이 필요합니다.
- **`SQLExpressions`** 클래스를 사용하여 다양한 SQL 함수를 활용할 수 있습니다.
- 함수 사용 시 타입 변환에 주의하여야 하며, 필요한 경우 **`NumberTemplate`**, **`StringTemplate`** 등을 사용하여 타입을 맞춰줄 수 있습니다.