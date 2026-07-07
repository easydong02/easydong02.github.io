---
title: "[Spring Data JPA] 임베디드 타입"
date: 2023-12-07 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **임베디드 타입**을 **복합키(@EmbeddedId)** 용도로 활용하는 방법을 정리한다.

> **임베디드 타입**: 새로운 값 타입을 직접 정의해 사용하는 것. `int`, `String`처럼 **값 타입**이라는 점이 핵심이다.

---

## 1. 복합키를 임베디드로 묶기

여러 필드가 함께 기본키(복합키)를 이룰 때, 이 필드들을 하나의 객체로 묶어 **`@EmbeddedId`**로 지정한다.

```java
@Entity
@Table(name = "code")
public class Code {
    @EmbeddedId               // 복합키를 임베디드 객체로
    private CodeId id;

    @Column(name = "CD_NM", length = 150)
    private String cdNm;
    // ...
}
```

```java
@EqualsAndHashCode
@Embeddable                   // 임베디드 타입 선언
public class CodeId implements Serializable {   // Serializable 필수
    @Column(name = "HGRN_CD_ID") private String hgrnCdId;
    @Column(name = "HGRN_CD")    private String hgrnCd;
    @Column(name = "CD_ID")      private String cdId;
    @Column(name = "CD")         private String cd;
}
```

두 어노테이션이 반드시 필요하다.

| 어노테이션 | 이유 |
|------|------|
| **`@EqualsAndHashCode`** | **값에 의한 객체 비교**를 위해. 없으면 값이 같아도 `new` 주소가 달라 비교 불가 |
| **`Serializable`** | JPA가 엔티티를 **영속화·캐시할 때 직렬화**를 지원해야 함 |

---

## 2. 복합키로 조회하기

```java
public interface CodeRepository extends JpaRepository<Code, CodeId>, CustomCodeRepository {
    Optional<Code> findById(CodeId id);   // 복합키 객체로 조회
}
```

**복합키 안의 특정 필드로 조회**하려면?

```java
Code findByIdCdId(String cdId);   // id 객체 안의 cdId 필드로 접근
```

> 💡 `findByIdCdId`는 "`id`(임베디드) 안의 `cdId` 필드로 찾기"를 의미한다. 임베디드 객체 안의 필드도 **중첩된 이름**으로 접근할 수 있다.

---

## 📝 정리

```
임베디드 타입 (복합키)
├─ @EmbeddedId   복합키를 임베디드 객체로 지정
├─ @Embeddable   임베디드 타입 선언
├─ 필수          @EqualsAndHashCode + Serializable
└─ 조회          findByIdCdId (임베디드 내부 필드 접근)
```

| 개념 | 한 줄 정의 |
|------|------|
| **@EmbeddedId** | 복합키를 하나의 객체로 |
| **@EqualsAndHashCode** | 값 기반 객체 비교 |
| **Serializable** | 직렬화 지원(영속화·캐시) |

복합키를 임베디드로 묶으면 여러 필드를 하나의 식별자 객체로 다룰 수 있다. 이때 **`@EqualsAndHashCode`와 `Serializable`**은 선택이 아니라 필수라는 점을 기억하자.
