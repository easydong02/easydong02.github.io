---
title: "[Spring Data JPA] 공통 인터페이스"
date: 2023-12-06 00:00:00 +0900
categories: [Database, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

# 공통인터페이스

## JPA VS Data JPA

---

기존의 JPA에서는 @Repository 어노테이션과 엔티티 매니저를 가져와서 작업을 했다. 하지만 Spring Data Jpa에서는 그냥 JpaRepository인터페이스를 상속받은 인터페이스로 작업이 가능하다.

**기존 JPA**

```java
@Repository
public class MemberJpaRepository {

    @PersistenceContext
    private EntityManager em;

    public Member save(Member member){
        em.persist(member);
        return member;
    }

    public void delete(Member member){
        em.remove(member);
    }

    public List<Member> findAll(){
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }

    public Optional<Member> findById(Long id){
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    public long count(){
        return em.createQuery("select count(m) from Member m",Long.class).getSingleResult();
    }

    public Member find(Long id){
        return em.find(Member.class, id);
    }
}
```

**Spring Data Jpa**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

이게 끝이다! 결국 인터페이스 안에 있는 추상 메서드 뿐인데 어떻게 save(), findAll()등이 가능하지?

→ 이것은 Spring이 JpaRepository를 상속받은 인터페이스는 **자동으로 추상메서드들은 구체화** 시켜주기 때문이다!

테스트 코드로 확인해보자

**MemberRepositoryTest**

```java
@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Autowired
    TeamRepository teamRepository;

    @Test
    public void testMember(){
        Member member = new Member("memberB");
        Member savedMember = memberRepository.save(member);

        Member findMember = memberRepository.findById(savedMember.getId()).get();

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember).isEqualTo(member);
    }
```
