---
title: "[Querydsl] Where 파라미터"
date: 2024-01-19 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, whereparameter]
render_with_liquid: false
future: true
---
# **QueryDSL - Where 파라미터: 동적 쿼리를 위한 지름길**

안녕하세요! 오늘은 QueryDSL에서 제공하는 **`where`** 파라미터에 대해 알아보겠습니다. **`where`** 파라미터는 동적인 쿼리를 작성할 때 유용한 기능으로, 다양한 상황에서 활용할 수 있습니다. 지금부터는 **`where`** 파라미터의 다양한 활용법을 예시 코드와 함께 살펴보겠습니다.

## ✅**Where 파라미터란?**

---

**`where`** 파라미터는 쿼리에서 조건을 동적으로 추가하거나 제거할 수 있는 기능입니다. 이를 통해 동적인 쿼리를 간편하게 작성할 수 있습니다.

## ✅**단일 조건 사용하기**

---

```java
javaCopy code
public List<Member> findMembersByCondition(String name, Integer age) {
    BooleanBuilder builder = new BooleanBuilder();

    if (name != null) {
        builder.and(member.name.eq(name));
    }

    if (age != null) {
        builder.and(member.age.eq(age));
    }

    return new JPAQueryFactory(entityManager)
        .selectFrom(member)
        .where(builder)
        .fetch();
}

```

위 코드에서는 **`BooleanBuilder`**를 사용하여 동적으로 조건을 추가하고 있습니다. **`name`**이나 **`age`**가 **`null`**이 아닐 경우에 해당 조건을 **`builder`**에 추가합니다.

## ✅**다중 조건 사용하기**

---

```java
javaCopy code
public List<Member> findMembersByConditions(String name, Integer age, String teamName) {
    return new JPAQueryFactory(entityManager)
        .selectFrom(member)
        .where(
            eqName(name),
            eqAge(age),
            eqTeamName(teamName)
        )
        .fetch();
}

private BooleanExpression eqName(String name) {
    return name != null ? member.name.eq(name) : null;
}

private BooleanExpression eqAge(Integer age) {
    return age != null ? member.age.eq(age) : null;
}

private BooleanExpression eqTeamName(String teamName) {
    return teamName != null ? member.team.name.eq(teamName) : null;
}

```

위 코드에서는 각 조건을 별도의 메서드로 분리하여 가독성을 높였습니다. 이러한 방식을 통해 다양한 조건을 추가하거나 변경할 수 있습니다.

## ✅**조건의 조합 사용하기**

---

```java
javaCopy code
public List<Member> findMembersByCombinedConditions(String name, Integer age, String teamName) {
    return new JPAQueryFactory(entityManager)
        .selectFrom(member)
        .where(
            eqNameOrTeamName(name, teamName),
            eqAge(age)
        )
        .fetch();
}

private BooleanExpression eqNameOrTeamName(String name, String teamName) {
    return name != null ? member.name.eq(name) : member.team.name.eq(teamName);
}

```

여러 조건을 조합하여 사용할 수도 있습니다. 위 코드에서는 이름 또는 팀 이름이 일치하는 경우를 찾아옵니다.

## 📌**주의사항과 팁**

---

- **`BooleanBuilder`**나 **`BooleanExpression`**을 사용하여 동적인 조건을 추가할 수 있습니다.
- 필요에 따라 조건을 메서드로 분리하여 가독성을 높일 수 있습니다.
- **`BooleanBuilder`**를 사용하면 여러 조건을 논리적으로 조합할 수 있습니다.