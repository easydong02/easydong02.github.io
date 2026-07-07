---
title: "[Spring Data JPA] 갑과 DTO 가져오기"
date: 2023-12-09 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 `@Query`로 **특정 값(단일 필드)**과 **DTO**를 조회하는 방법을 정리한다. 엔티티 전체가 아니라 필요한 데이터만 뽑아올 때 유용하다.

---

## 1. 값 가져오기 (단일 필드)

엔티티가 아니라 특정 컬럼 값만 List로 조회한다.

```java
@Query("select m.username from Member m")
List<String> findUsernameList();
```

```java
List<String> result = memberRepository.findUsernameList();
// AAA
// BBB
```

---

## 2. DTO 가져오기

여러 엔티티에서 필요한 값만 골라 **DTO**로 받는다. 먼저 DTO를 만든다.

```java
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

**JPQL의 `new`**로 DTO 경로를 지정해 조회한다.

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t")
List<MemberDto> findMemberDto();
```

> 💡 JPQL에서는 **`new 전체경로.Dto클래스(...)`**로 DTO를 만든다. 이때 파라미터에 맞는 **생성자가 DTO에 미리 선언**되어 있어야 한다.

```java
List<MemberDto> result = memberRepository.findMemberDto();
// MemberDto(id=1, username=AAA, teamname=teamA)
```

| 조회 대상 | 방법 |
|------|------|
| 단일 값 | `select m.username` → `List<String>` |
| DTO | `select new 경로.Dto(...)` → `List<Dto>` |

---

## 📝 정리

```
값 & DTO 조회
├─ 값     @Query("select m.필드") → List<타입>
└─ DTO    @Query("select new 경로.Dto(...)") + 생성자 필요
```

| 개념 | 한 줄 정의 |
|------|------|
| **단일 값 조회** | 특정 컬럼만 뽑기 |
| **DTO 조회** | `new` 문법으로 필요한 값만 조립 |

엔티티 전체가 아니라 **필요한 데이터만** 조회하면 성능과 관심사 분리에 유리하다. DTO 조회 시에는 **JPQL의 `new`**와 **DTO 생성자**가 짝을 이뤄야 한다는 점을 기억하자.
