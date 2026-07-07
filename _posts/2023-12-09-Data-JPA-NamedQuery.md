---
title: "[Spring Data JPA] 네임드 쿼리"
date: 2023-12-09 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, namedquery]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **네임드 쿼리(Named Query)**를 순수 JPA와 Spring Data JPA에서 각각 사용하는 법을 비교한다.

> **네임드 쿼리란?**
> 엔티티에 **쿼리문을 이름과 함께 미리 저장**해두고, 레포지토리에서 그 이름으로 바로 꺼내 쓰는 기능.

---

## 1. 엔티티에 쿼리 등록 (@NamedQuery)

```java
@NamedQuery(
    name = "Member.findByUsername",
    query = "select m from Member m where m.username = :username"
)
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String username;
    private int age;
    // ...
}
```

---

## 2. 순수 JPA — createNamedQuery

```java
public List<Member> findByUsername(String username) {
    return em.createNamedQuery("Member.findByUsername", Member.class)
        .setParameter("username", username)
        .getResultList();
}
```

`createNamedQuery`로 이름을 지정해 미리 저장한 쿼리를 실행한다.

---

## 3. Spring Data JPA — @Query

```java
@Query(name = "Member.findByUsername")
List<Member> findByUsername(@Param("username") String username);
```

| 방식 | 실행 방법 |
|------|------|
| 순수 JPA | `em.createNamedQuery("이름")` + `setParameter` |
| **Data JPA** | `@Query(name="이름")` + `@Param` |

> 💡 Spring Data JPA는 **`@Query`** 어노테이션으로 저장한 이름의 쿼리를 꺼내고, **`@Param`**으로 파라미터를 동적으로 바인딩한다. (사실 메소드 이름이 `엔티티.메소드명` 규칙과 맞으면 `@Query` 없이도 네임드 쿼리를 자동으로 찾는다.)

---

## 📝 정리

```
네임드 쿼리
├─ 등록    엔티티에 @NamedQuery(name, query)
├─ 순수JPA  em.createNamedQuery("이름")
└─ Data JPA @Query(name="이름") + @Param
```

| 개념 | 한 줄 정의 |
|------|------|
| **@NamedQuery** | 이름 붙인 쿼리를 엔티티에 저장 |
| **@Query(name)** | 저장된 네임드 쿼리 사용 |
| **@Param** | 파라미터 바인딩 |

네임드 쿼리는 "쿼리를 미리 이름으로 저장"하는 방식이다. Spring Data JPA에서는 `@Query`와 `@Param`으로 간결하게 활용할 수 있다.
