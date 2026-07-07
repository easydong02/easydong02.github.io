---
title: "[Querydsl] sql함수(Sql functions)"
date: 2024-01-18 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, sqlfunction]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL에서 **SQL 함수**를 활용하는 법을 정리한다.

> **SQL Function이란?** DB가 제공하는 다양한 함수를 QueryDSL에서 쓸 수 있게 해주는 기능. **`SQLExpressions`** 클래스로 활용한다.

---

## 1. 문자열 함수

```java
// concat: 문자열 합치기 → "이름(나이)"
SQLExpressions.concat(member.name, "(", member.age, ")")

// lower: 소문자 변환
SQLExpressions.lower(member.name)
```

## 2. 날짜 함수

```java
SQLExpressions.currentDate()              // 현재 날짜
SQLExpressions.addDays(member.joinDate, 7)  // joinDate + 7일
```

## 3. 수학 함수

```java
SQLExpressions.sqrt(member.age.doubleValue())  // 제곱근
SQLExpressions.round(member.height)            // 반올림
```

## 4. coalesce — NULL 처리

```java
SQLExpressions.coalesce(member.nickname, "닉네임 없음")   // null이면 기본값
```

주요 함수를 정리하면:

| 분류 | 함수 |
|------|------|
| 문자열 | `concat`, `lower` |
| 날짜 | `currentDate`, `addDays` |
| 수학 | `sqrt`, `round` |
| NULL | `coalesce`(null → 기본값) |

---

## 📌 주의사항과 팁

> ⚠️ SQL 함수는 **DB 종류에 따라 지원 여부가 다르다.** 함수 사용 시 타입 변환에 주의하며, 필요하면 **`NumberTemplate`·`StringTemplate`**으로 타입을 맞춘다.

---

## 📝 정리

```
QueryDSL SQL 함수 (SQLExpressions)
├─ 문자열   concat, lower
├─ 날짜     currentDate, addDays
├─ 수학     sqrt, round
└─ NULL     coalesce
```

| 개념 | 한 줄 정의 |
|------|------|
| **SQLExpressions** | SQL 함수 활용 클래스 |
| **coalesce** | null을 기본값으로 |
| **Template** | 미지원 함수를 직접 정의 |

SQL 함수를 QueryDSL에서 쓰면 데이터를 유연하게 가공할 수 있다. 다만 **DB마다 지원 함수가 다르다**는 점, 타입 변환에 유의해야 한다는 점을 기억하자.
