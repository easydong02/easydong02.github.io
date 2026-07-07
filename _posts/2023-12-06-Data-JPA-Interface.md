---
title: "[Spring Data JPA] 공통 인터페이스"
date: 2023-12-06 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 순수 JPA와 Spring Data JPA의 **Repository 코드 차이**를 비교한다. Spring Data JPA가 얼마나 많은 코드를 줄여주는지 확인해보자.

---

## 1. 순수 JPA — 직접 구현

기존 JPA는 `@Repository`와 `EntityManager`로 CRUD를 **일일이 구현**한다.

```java
@Repository
public class MemberJpaRepository {
    @PersistenceContext
    private EntityManager em;

    public Member save(Member member) { em.persist(member); return member; }
    public void delete(Member member) { em.remove(member); }
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }
    public Optional<Member> findById(Long id) {
        return Optional.ofNullable(em.find(Member.class, id));
    }
    public long count() {
        return em.createQuery("select count(m) from Member m", Long.class).getSingleResult();
    }
}
```

---

## 2. Spring Data JPA — 인터페이스만

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

> 💡 **이게 끝이다!** 추상 메소드조차 없는데 어떻게 `save()`, `findAll()`이 동작할까? **Spring이 `JpaRepository`를 상속한 인터페이스의 구현체를 자동으로 만들어주기** 때문이다. (프록시로 구현 클래스를 생성해 주입)

```
순수 JPA (수십 줄)         Spring Data JPA (1줄)
@Repository class {        interface extends JpaRepository {
  save, delete, findAll,       }  ← 구현은 스프링이 자동 생성
  findById, count ...
}
```

| 항목 | 순수 JPA | Spring Data JPA |
|------|------|------|
| 구현 | 직접 작성 | **자동 생성** |
| 코드량 | 수십 줄 | **1줄** |
| EntityManager | 직접 다룸 | 감춰짐 |

---

## 3. 테스트로 확인

```java
@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {
    @Autowired MemberRepository memberRepository;

    @Test
    public void testMember() {
        Member member = new Member("memberB");
        Member savedMember = memberRepository.save(member);        // 자동 구현된 save
        Member findMember = memberRepository.findById(savedMember.getId()).get();
        assertThat(findMember).isEqualTo(member);
    }
}
```

빈 인터페이스만으로 `save`, `findById`가 정상 동작한다.

---

## 📝 정리

```
JPA vs Spring Data JPA
├─ 순수 JPA    @Repository + EntityManager로 직접 구현
├─ Data JPA    JpaRepository 상속만 (스프링이 자동 구현)
└─ 원리        상속 인터페이스의 구현체를 프록시로 자동 생성
```

| 개념 | 한 줄 정의 |
|------|------|
| **JpaRepository** | 공통 CRUD를 제공하는 인터페이스 |
| **자동 구현** | 스프링이 구현체를 프록시로 생성 |

핵심은 **"Spring Data JPA가 인터페이스 구현체를 자동으로 만들어준다"**는 것이다. 순수 JPA의 반복적인 Repository 코드가 단 한 줄로 사라지는 것이 이 프레임워크의 가장 큰 매력이다.
