---
title: [Spring Data JPA] 갑과 DTO 가져오기
date: 2023-12-09 00:00:00 +0900
categories: [Database, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

# 값과 DTO 가져오기

## 값 가져오기

---

**MemberRepository.java**

```java
@Query("select m.username from Member m")
    List<String> findUsernameList();
```

MemberRepositoryTest.java

```java
@Test
    public void findUsernameList(){
        Member m1 = new Member("AAA", 10);
        Member m2 = new Member("BBB", 20);

        memberRepository.save(m1);
        memberRepository.save(m2);

        List<String> result = memberRepository.findUsernameList();

        for(String name: result){
            System.out.println(name);
        }
    }

---

AAA
BBB
```

## DTO가져오기

---

**MemberDto.java**

```java
package study.datajpa.dto;

import lombok.Data;

@Data
public class MemberDto {

    private Long id;
    private String username;
    private String teamname;

    public MemberDto(Long id, String username, String teamname) {
        this.id = id;
        this.username = username;
        this.teamname = teamname;
    }
}
```

**MemberRepository.java**

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();
```

→ JPQL에서는 new 로 dto의 경로까지 해서 알려준다. 여기서 저 파라미터로 들어간 생성자가 이미 선언되어 있어야한다.

**MemberRepositoryTeat.java**

```java
@Test
    public void findMemberDto(){
        Team t1 = new Team("teamA");
        teamRepository.save(t1);

        Member m1 = new Member("AAA", 10);
        m1.setTeam(t1);
        memberRepository.save(m1);

        List<MemberDto> result = memberRepository.findMemberDto();

        for(MemberDto memberDto: result){
            System.out.println(memberDto);
        }
    }

---

MemberDto(id=1, username=AAA, teamname=teamA)
```
