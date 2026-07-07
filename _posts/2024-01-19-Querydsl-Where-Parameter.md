---
title: "[Querydsl] Where 파라미터"
date: 2024-01-19 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, whereparameter]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL로 **동적 쿼리**를 작성하는 두 방식을 정리한다. 조건을 상황에 따라 붙였다 뗐다 하는 것이 핵심이다.

| 방식 | 도구 |
|------|------|
| **BooleanBuilder** | 조건을 하나의 빌더에 누적 |
| **Where 다중 파라미터** | `BooleanExpression` 메소드를 나열 (null이면 무시) |

---

## 1. BooleanBuilder

```java
public List<Member> findMembersByCondition(String name, Integer age) {
    BooleanBuilder builder = new BooleanBuilder();
    if (name != null) builder.and(member.name.eq(name));   // 있을 때만 추가
    if (age != null)  builder.and(member.age.eq(age));

    return queryFactory
        .selectFrom(member)
        .where(builder)
        .fetch();
}
```

> 💡 `if`로 값이 있을 때만 조건을 `and`로 누적한다. `null`인 파라미터는 조건에서 빠진다.

## 2. Where 다중 파라미터 (권장)

`where()`에 여러 `BooleanExpression`을 나열하면 **null인 것은 자동으로 무시**된다.

```java
public List<Member> findMembers(String name, Integer age, String teamName) {
    return queryFactory
        .selectFrom(member)
        .where(eqName(name), eqAge(age), eqTeamName(teamName))   // null은 무시됨
        .fetch();
}

private BooleanExpression eqName(String name) {
    return name != null ? member.name.eq(name) : null;   // null 반환 시 조건 제외
}
private BooleanExpression eqAge(Integer age) {
    return age != null ? member.age.eq(age) : null;
}
```

> 💡 조건을 **메소드로 분리**하면 재사용성과 가독성이 좋아진다. `where(A, B, C)`는 A AND B AND C이며, **null인 인자는 건너뛴다.**

## 3. 조건 조합

메소드를 조합해 더 복잡한 조건도 만든다.

```java
private BooleanExpression eqNameOrTeamName(String name, String teamName) {
    return name != null ? member.name.eq(name) : member.team.name.eq(teamName);
}
```

---

## 📝 정리

```
QueryDSL 동적 쿼리
├─ BooleanBuilder   if로 조건 누적 (builder.and)
├─ Where 다중       where(A, B, C) — null 자동 무시
└─ 조합             메소드 분리로 재사용·가독성↑
```

| 개념 | 한 줄 정의 |
|------|------|
| **BooleanBuilder** | 조건을 누적하는 빌더 |
| **BooleanExpression** | 조건 하나(null이면 무시) |
| **Where 다중 파라미터** | 조건 메소드를 나열 |

동적 쿼리는 QueryDSL의 강력한 장점이다. 특히 **Where 다중 파라미터** 방식은 조건을 메소드로 분리해 **재사용·가독성**이 좋아, `BooleanBuilder`보다 실무에서 선호된다.
