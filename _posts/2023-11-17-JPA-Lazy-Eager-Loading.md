---
title: 즉시(Eager)로딩 지연(Lazy)로딩
date: 2023-11-16 00:00:00 +0900
categories: [Database, JPA]
tags: [jpa, lazy, eager]
render_with_liquid: false
future: true
---

# 즉시로딩 지연로딩

## 지연로딩

---

- fetch 옵션을 Lazy했을 때 가져온 객체는 프록시 객체로 가져와서 실제로 사용할 때(+컨텍스트 관리도 받을때) 초기화 되어서 실제 entity와 연결된다.

```java
@ManyToOne(fetch = FetchType.LAZY)//Lazy로 하면 Team은 프록시 객체로 준다.
@JoinColumn(insertable = false, updatable = false, name = "TEAM_ID") //읽기 전용 필드 옵션
private Team team;

Member member = new Member();
member.setUsername("John");

Team team = new Team();
team.setName("teamA");

em.persist(member);

team.getMembers().add(member);

em.persist(team);

em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());

System.out.println("m= "+findMember.getTeam().getClass());

---

m= class hellojpa.Team$HibernateProxy$Flm1Zq0H
```

- 따라서 자주 같이 사용하면 fetch 옵션을 eager로 줘서 즉시로딩으로 같이 쓸 수 있다.

## 즉시로딩

---

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(insertable = false, updatable = false, name = "TEAM_ID") //읽기 전용 필드 옵션
private Team team;

Member member = new Member();
member.setUsername("John");

Team team = new Team();
team.setName("teamA");

em.persist(member);

team.getMembers().add(member);

em.persist(team);

em.flush();
em.clear();

Member findMember = em.find(Member.class, member.getId());

System.out.println("m= "+findMember.getTeam().getClass());

---

m = class hellojpa.Team
```

## 프록시와 즉시로딩 주의

---

- 실무에선 가급적 **지연로딩만** 사용!
- 즉시 로딩은 SQL문제가 발생할 수 있기 때문.
- 즉시 로딩은 JPQL에서 **N+1문제**를 일으킬 수 있기 때문.(조인문으로 가져온 것들에서 관련된 객체들 전부 가져오기 때문이다.)
- @ManyToOne, @OneToOne할 때는 **디폴트가 즉시로딩이기 때문에 지연로딩으로** 아예 해버려야 한다.

## 지연 로딩 활용

---

- Member와 Team은 **자주** 함께 사용 → **즉시 로딩**
- Member와 Order는 **가끔** 사용 → **지연 로딩**
- Order와 Product는 **자주** 함께 사용 → **즉시 로딩**
- **JPQL fetch 조인**을 사용해서 즉시 로딩을 사용해라(기본으로 다 Lazy로 하고)
