---
title: "[Spring Data JPA] 임베디드 타입"
date: 2023-12-07 00:00:00 +0900
categories: [Database, Spring-Data-JPA]
tags: [springdatajpa]
render_with_liquid: false
future: true
---

# 임베디드 타입

## 임베디드 객체란?

---

새로운 값 타입을 직접 정의해서 사용할 수 있는데, JPA에서는 이것을 임베디드 타입(embedded type)이라 합니다.

중요한 것은 직접 정의한 임베디드 타입도 `int, String`처럼 값 타입이라는 것입니다.

## 예제

---

**Code.java**

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@Table(name = "code")
public class Code {

    @EmbeddedId
    private CodeId id;

    @Column(name = "CD_NM", length = 150)
    private String cdNm;

    @Column(name = "RGS_EMP_NO", length = 32)
    private String rgsEmpNo;

```

여기서는 프라이머리키가 4개 필드의 복합키로 되어있습니다. 따라서 이 4개의 필드를 묶어서 하나의 객체로 만들었고 그것을 @EmbeddedId 어노테이션으로 설정했습니다.

**CodeId.java**

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
@Embeddable
public class CodeId implements Serializable {

    @Column(name = "HGRN_CD_ID", length = 16)
    private String hgrnCdId;

    @Column(name = "HGRN_CD", length = 16)
    private String hgrnCd;

    @Column(name = "CD_ID", length = 16)
    private String cdId;

    @Column(name = "CD", length = 16)
    private String cd;
}
```

@Embeddable 어노테이션으로 임베디드 타입이란 것을 설정한 뒤 @EqualAndHashCode와 Serializable을 implements 해줬습니다.

**@EqualAndHashCode?**

→ **`equals()`** 메서드는 두 객체가 같은지 여부를 판단하는데 사용되며, **`hashCode()`** 메서드는 객체의 해시코드 값을 반환합니다. 이 두 메서드는 Java에서 객체 동등성과 해시맵과 같은 컬렉션을 사용할 때 매우 중요합니다. 그러므로 이러한 메서드들이 제대로 구현되어야 합니다. 이것을 하지않으면 객체간 비교할 때 값이 다 같아도 new에 의한 주소값이 다르므로 객체비교를 하기 힘듭니다. 따라서 **`값에 의한 객체 비교`**를 위해서 사용합니다

**Serializable?**

→ **`Serializable`** 인터페이스는 Java 직렬화를 지원하기 위한 인터페이스입니다. 객체를 직렬화하면, 이를 파일에 저장하거나 네트워크를 통해 전송할 수 있게 됩니다. JPA에서 엔티티를 다룰 때, 특히 객체를 영속화하거나 캐시하기 위해 직렬화를 지원하는 것이 중요합니다. 따라서 **`Serializable`** 인터페이스를 구현하는 것은 JPA에서 객체의 직렬화를 지원하고 객체를 영속화하는 과정에서 문제가 발생하지 않도록 하는 데 도움을 줍니다.

**CodeRepository.java**

```java
public interface CodeRepository extends JpaRepository<Code, CodeId>, CustomCodeRepository {

    Optional<Code> findById(CodeId id);
}
```

여기선 JpaRepository를 상속받습니다. 그리고 복합키로 쓰이는 CodeId객체를 작성해줍니다.

이러면 복합키 객체인 CodeId로 찾기가 가능합니다.

만약 여기서 CodeId 안에 있는 cdId필드로 찾고싶다면?

```java
Code findByIdCdId(String cdId);
```

이렇게 해주면 되겠죠? 객체 안에 있는 필드이기 때문에 이렇게 접근이 가능하답니다!
