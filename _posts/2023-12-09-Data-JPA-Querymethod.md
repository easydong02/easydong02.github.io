---
title: 쿼리 메소드
date: 2023-12-09 00:00:00 +0900
categories: [Database, Spring-Data-JPA]
tags: [springdatajpa, querymethod]
render_with_liquid: false
future: true
---

# 쿼리메소드?

---

→ 쿼리 메소드란 spring data jpa에서 지원하는 레포지토리의 기능 중 하나로 레포지토리 안에 있는 메소드의 정해진 형식의 이름으로써 그 기능을 하게 된다. 여기에선 이해를 돕기 위해 기본 JPA를 사용했을 때와 data jpa의 쿼리메소드 기능을 사용했을 때의 차이를 보여준다.

## 기본 JPA

---

**MemberJpaRepository.java**

```java
public List<Member> findByUsernameAndAgeGreaterThan(String username, int age){
        return em.createQuery("select m from Member m where m.username= :username and m.age > :age")
                .setParameter("username", username)
                .setParameter("age",age)
                .getResultList();
    }
```

**MemberJpaRepositoryTest.java**

```java
@Test
    public void findByUsernameAndAgeGreaterThan(){
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberJpaRepository.save(m1);
        memberJpaRepository.save(m2);

        List<Member> result = memberJpaRepository.findByUsernameAndAgeGreaterThan("AAA", 15);

        assertThat(result.get(0).getUsername()).isEqualTo(m2.getUsername());
        assertThat(result.get(0).getAge()).isEqualTo(m2.getAge());
        assertThat(result.size()).isEqualTo(1);
    }
```

→ 기본 JPA를 사용했을 때는 findByUsernameAndAgeGreaterThan이름의 메소드를 만들고 엔티티 매니저를 사용하여 원하는 로직을 구현했다. 물론 테스트코드에서 잘 돌아간다.

## **Spring Data JPA**

---

**MemberRepository.java**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

**MemberRepositoryTest.java**

```java
@Test
    public void findByUsernameAndAgeGreaterThan(){
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("AAA", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<Member> result = memberRepository.findByUsernameAndAgeGreaterThan("AAA", 15);

        assertThat(result.get(0).getUsername()).isEqualTo(m2.getUsername());
        assertThat(result.get(0).getAge()).isEqualTo(m2.getAge());
        assertThat(result.size()).isEqualTo(1);
    }
```

→ spring data jpa에서는 실제 구현하지 않더라도 레포지토리 인터페이스에서 정해진 형식에 따라서 이름을 지은 메소드가 실제로 그 기능을 한다는 것이다.
