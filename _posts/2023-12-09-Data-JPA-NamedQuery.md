---
title: "[Spring Data JPA] 네임드 쿼리"
date: 2023-12-09 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, namedquery]
render_with_liquid: false
future: true
---

# 네임드 쿼리

## 네임드 쿼리?

---

→ 네임드 쿼리란 엔티티에서 미리 정한 쿼리문을 저장했다가 레포지토리에서 그것을 바로 쓸 수 있도록 이름을 지정하여 쿼리문을 저장하는 기능이다.

## JPA vs Spring data JPA

---

**Member.java**

```java
@NamedQuery(
        name ="Member.findByUsername",
        query = "select m from Member m where m.username = :username"
)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "MEMBER_ID")
    private Long id;
    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public Member(String username) {
        this.username = username;
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        this.team = team;
    }

    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public void changeTeam(Team team){
        this.team = team;
        team.getMembers().add(this);
    }
}
```

→ 미리 @NamedQuery 어노테이션을 통해서 쿼리문을 만들어놓는다.

### JPA

---

**MemberJpaRepository.java**

```java
public List<Member> findByUsername(String username){
        return em.createNamedQuery("Member.findByUsername", Member.class)
                .setParameter("username", username)
                .getResultList();
    }
```

**MemberJpaRepositoryTest.java**

```java
@Test
    public void testNamedQuery(){
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberJpaRepository.save(m1);
        memberJpaRepository.save(m2);

        List<Member> result = memberJpaRepository.findByUsername(m2.getUsername());
        Member findMember = result.get(0);
        assertThat(findMember).isEqualTo(m2);
    }
```

→ createNamedQuery라는 메소드를 사용하여 미리 이름으로써 저장한 쿼리문을 가져와서 작동시킨다.

### Spring Data JPA

---

**MemberRepository.java**

```java
@Query(name = "Member.findByUsername")
    List<Member> findByUsername(@Param("username") String username);
```

**MemberRepositoryTest.java**

```java
@Test
    public void testNamedQuery(){
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> result = memberRepository.findByUsername(m2.getUsername());
        Member findMember = result.get(0);
        assertThat(findMember).isEqualTo(m2);
    }
```

→ 레포지토리에서 @Query라는 어노테이션을 통해서 저장한 이름의 쿼리문을 꺼내온다. 그리고 @Param 어노테이션으로 동적으로 파라미터를 설정한다.
