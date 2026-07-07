---
title: "[JPA] 즉시(Eager)로딩 지연(Lazy)로딩"
date: 2023-11-16 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa, lazy, eager]
render_with_liquid: false
future: true
---

## 📌 들어가며

연관된 엔티티를 **언제 가져올지**를 결정하는 것이 로딩 전략이다. 지금 당장 함께 가져올지(**즉시, Eager**), 실제로 쓸 때 가져올지(**지연, Lazy**)를 `fetch` 옵션으로 정한다.

| 전략 | fetch 옵션 | 동작 |
|------|:---:|------|
| **지연 로딩** | `FetchType.LAZY` | 연관 객체를 **프록시**로 두고, 실제 사용 시 초기화 |
| **즉시 로딩** | `FetchType.EAGER` | 조회 시점에 연관 객체를 **함께** 가져옴 |

---

## 1. 지연 로딩 (Lazy)

```java
@ManyToOne(fetch = FetchType.LAZY)   // Team을 프록시로 반환
@JoinColumn(name = "TEAM_ID")
private Team team;
```

```java
Member findMember = em.find(Member.class, member.getId());
System.out.println(findMember.getTeam().getClass());
// → class hellojpa.Team$HibernateProxy$Flm1Zq0H   (프록시!)
```

> 💡 Lazy로 가져온 `team`은 **프록시 객체**다. 실제로 `team`을 사용하는 순간(영속성 컨텍스트 관리 하에서) 초기화되어 실제 엔티티와 연결된다.

---

## 2. 즉시 로딩 (Eager)

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "TEAM_ID")
private Team team;
```

```java
Member findMember = em.find(Member.class, member.getId());
System.out.println(findMember.getTeam().getClass());
// → class hellojpa.Team   (실제 엔티티!)
```

---

## 3. ⚠️ 실무 주의사항

> ⚠️ **실무에서는 가급적 지연 로딩(Lazy)만 사용하자.**

| 문제 | 설명 |
|------|------|
| **예측 불가한 SQL** | 즉시 로딩은 의도치 않은 조인 SQL을 유발 |
| **N+1 문제** | JPQL에서 연관 객체를 하나씩 추가 조회 → 쿼리 폭증 |
| **기본값 함정** | `@ManyToOne`, `@OneToOne`은 **기본이 즉시 로딩** → 명시적으로 Lazy로! |

> 💡 즉시 로딩이 꼭 필요할 때는 필드 옵션이 아니라 **JPQL의 fetch 조인**을 사용한다. (기본은 다 Lazy로 두고, 필요할 때만 fetch join)

---

## 📝 정리

```
Eager vs Lazy 로딩
├─ Lazy    프록시로 두고 사용 시 초기화 (권장)
├─ Eager   조회 시 함께 로딩 (N+1 위험)
├─ 함정    @ManyToOne/@OneToOne은 기본이 Eager → Lazy로
└─ 대안    필요 시 JPQL fetch join
```

| 개념 | 한 줄 정의 |
|------|------|
| **지연 로딩** | 실제 사용 시점에 로딩(프록시) |
| **즉시 로딩** | 조회 시 함께 로딩 |
| **N+1 문제** | 연관 객체를 반복 조회해 쿼리 폭증 |

로딩 전략의 결론은 명확하다. **"기본은 전부 Lazy, 필요할 때만 fetch join"**. 특히 `@ManyToOne`·`@OneToOne`의 기본값이 Eager라는 함정을 기억하고, 항상 명시적으로 Lazy를 지정하자.
