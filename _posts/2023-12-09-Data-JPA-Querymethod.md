---
title: "[Spring Data JPA] 쿼리 메소드"
date: 2023-12-09 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, querymethod]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 Spring Data JPA의 **쿼리 메소드**를 정리한다.

> **쿼리 메소드란?**
> 레포지토리 메소드의 **이름을 정해진 형식으로 짓기만 하면** 그 기능을 하는 것. 순수 JPA와 비교하면 그 편리함이 극명하다.

---

## 1. 순수 JPA — 직접 쿼리 작성

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
    return em.createQuery(
            "select m from Member m where m.username = :username and m.age > :age")
        .setParameter("username", username)
        .setParameter("age", age)
        .getResultList();
}
```

메소드 이름을 짓고, **JPQL을 직접 작성**하고, 파라미터를 바인딩해야 한다.

---

## 2. Spring Data JPA — 이름만

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

> 💡 **구현이 전혀 없다!** 메소드 이름을 규칙에 맞게 지으면, 스프링이 이름을 분석해 **자동으로 쿼리를 만들어** 실행한다.

```
findBy  Username  And  Age  GreaterThan
  │        │       │    │        │
select   where   AND  age   age > ?
```

| 방식 | 쿼리 작성 |
|------|------|
| 순수 JPA | JPQL 직접 작성 + 파라미터 바인딩 |
| **쿼리 메소드** | **메소드 이름만** (자동 생성) |

---

## 3. 테스트 (동일하게 동작)

```java
@Test
public void findByUsernameAndAgeGreaterThan() {
    Member m1 = new Member("AAA", 10);
    Member m2 = new Member("AAA", 20);
    memberRepository.save(m1);
    memberRepository.save(m2);

    // "AAA"이면서 age > 15인 것 → m2만
    List<Member> result = memberRepository.findByUsernameAndAgeGreaterThan("AAA", 15);

    assertThat(result.get(0).getAge()).isEqualTo(m2.getAge());
    assertThat(result.size()).isEqualTo(1);
}
```

순수 JPA로 구현한 것과 **완전히 동일하게** 동작한다.

---

## 📝 정리

```
쿼리 메소드
├─ 순수 JPA   메소드 + JPQL 직접 작성
├─ 쿼리 메소드  메소드 이름만 → 쿼리 자동 생성
└─ 규칙       findBy + 필드 + 조건(And/GreaterThan 등)
```

| 키워드 | 의미 |
|:---:|------|
| `findBy` | select |
| `And` | AND 조건 |
| `GreaterThan` | `>` |

쿼리 메소드는 "메소드 이름이 곧 쿼리"라는 발상이다. 간단한 조회는 JPQL을 한 줄도 안 쓰고 **이름만으로** 해결되니, 반복적인 조회 코드가 크게 줄어든다.
