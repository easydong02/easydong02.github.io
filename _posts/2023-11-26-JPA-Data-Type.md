---
title: "[JPA] 값 타입"
date: 2023-11-26 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa, hibernate]
render_with_liquid: false
future: true
---

## 📌 들어가며

JPA의 데이터는 크게 **엔티티 타입**과 **값 타입**으로 나뉜다. 이번 글에서는 값 타입(기본값·임베디드·컬렉션)을 정리한다.

| 타입 | 특징 |
|------|------|
| **엔티티 타입** | `@Entity`, **식별자로 추적 가능** (변해도 추적) |
| **값 타입** | 식별자 없음, **변경 시 추적 불가** |

**값 타입 분류:**

```
값 타입
├─ 기본값 타입     int, Integer, String
├─ 임베디드 타입   기본값을 모은 복합 타입 (@Embeddable)
└─ 컬렉션 값 타입  값 타입을 여러 개 (@ElementCollection)
```

---

## 1. 기본값 타입

> `int`, `long`, `String`처럼 단순히 값으로 쓰는 타입. **식별자가 없어 변경 시 추적 불가**하고, 생명주기가 엔티티에 의존한다.

> ⚠️ **값 타입은 공유하면 안 된다.** 값 복사(call by value)로 다뤄야 사이드 이펙트가 없다.

---

## 2. 임베디드 타입 (@Embeddable)

> 기본값 타입을 **모아서 만든 복합 타입**(예: 주소 = 도시+거리+우편번호). 새로운 값 타입을 사용자 정의로 만든다.

```java
@Embeddable          // 값 타입 정의하는 곳
public class Address { ... }

@Embedded            // 값 타입 사용하는 곳
private Address address;
```

> ⚠️ **기본 생성자 필수.** 그리고 여러 엔티티에서 **공유하면 위험**하다. 임베디드 값이 바뀌면 그것을 참조하는 **모든 엔티티의 값이 함께 바뀌기** 때문. 같은 값을 넣고 싶으면 **새 임베디드를 만들어 값 복사**하자.

| 장점 | 설명 |
|------|------|
| 재사용 | 여러 엔티티에서 사용 |
| 높은 응집도 | 관련 값을 하나로 묶음 |
| 의미 있는 메소드 | `Period.isWork()` 같은 메소드 |

> 💡 임베디드 타입을 써도 **매핑 테이블은 그대로**다. 클래스만 세밀하게 나뉠 뿐이다. (잘 설계한 ORM은 테이블보다 클래스 수가 많다) 같은 값 타입을 한 엔티티에서 여러 번 쓰면 컬럼명이 중복되므로 **`@AttributeOverride`**로 재정의한다.

---

## 3. 불변 객체로 만들기

> ⚠️ 값 타입의 공유 사고를 **원천 차단**하려면 **불변 객체**로 만든다. **생성자로만 값을 설정하고 Setter를 만들지 않으면** 된다.

---

## 4. 값 타입 비교

| 비교 | 방법 | 대상 |
|------|:---:|------|
| **동일성** | `==` | 참조값 비교 |
| **동등성** | `equals()` | 인스턴스 값 비교 |

> 💡 값 타입은 **`equals()`를 재정의해 동등성 비교**하는 게 좋다. 재정의 시 필드 직접 접근보다 **getter를 쓰자**(프록시 객체일 때 필드 직접 접근이 안 되기 때문).

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Address a = (Address) o;
    return Objects.equals(getCity(), a.getCity())
        && Objects.equals(getStreet(), a.getStreet())
        && Objects.equals(getZipcode(), a.getZipcode());
}
@Override
public int hashCode() { return Objects.hash(getCity(), getStreet(), getZipcode()); }
```

---

## 5. 값 타입 컬렉션

> 값 타입을 **하나 이상 저장**할 때 `@ElementCollection`, `@CollectionTable`을 쓴다. DB는 컬렉션을 한 테이블에 못 넣으므로 **별도 테이블**이 필요하다.

> 💡 컬렉션은 소유 엔티티가 생명주기를 정하므로, **Cascade + 고아 객체 제거**가 함께 있는 것과 같다. 지연 로딩이며, **수정할 때는 새 객체로 통째로 갈아 끼워야** 한다.

```java
// ✅ 새 객체로 교체
findMember.setAddress(new Address("newCity", addr.getStreet(), addr.getZipcode()));
// ❌ 절대 이렇게 하면 안 됨
findMember.getAddress().setCity("newCity");
```

> ⚠️ **값 타입 컬렉션의 제약**: 식별자가 없어 변경 추적이 어렵다. **변경이 발생하면 연관된 데이터를 전부 삭제하고 현재 값을 모두 다시 저장**한다(비효율). 기본키는 모든 컬럼을 묶어 구성(null·중복 불가).

```sql
-- 하나 바꿨는데 전부 삭제 후 재삽입!
delete from ADDRESS where MEMBER_ID=?
insert into ADDRESS (...) values (...)
insert into ADDRESS (...) values (...)
```

### 대안 — 일대다 관계

> 💡 **실무에서는 값 타입 컬렉션 대신 일대다(엔티티) 관계**를 고려한다. 값 타입을 엔티티로 승격시켜 `@OneToMany` + Cascade + orphanRemoval로 관리한다.

```java
// Member
@OneToMany(mappedBy = "member", cascade = CascadeType.ALL, orphanRemoval = true)
private List<AddressEntity> addressEntityList = new ArrayList<>();
```

---

## 📝 정리

```
JPA 값 타입
├─ 기본값    int/String, 추적 불가, 공유 금지
├─ 임베디드  @Embeddable/@Embedded, 불변으로 (Setter X)
├─ 비교      equals() 재정의 (getter 사용)
├─ 컬렉션    @ElementCollection, 변경 시 전체 삭제·재삽입
└─ 대안      실무는 일대다 엔티티 관계
```

| 개념 | 한 줄 정의 |
|------|------|
| **임베디드 타입** | 값을 모은 복합 값 타입 |
| **불변 객체** | Setter 없이 생성자로만 |
| **값 타입 컬렉션** | 비효율적(전체 재삽입) → 일대다 권장 |

값 타입의 핵심은 **"공유하지 말고, 불변으로 만들라"**다. 특히 값 타입 컬렉션은 변경 시 전체를 삭제·재삽입하는 비효율이 있어, 실무에서는 **일대다 엔티티 관계**로 풀어내는 것이 정석이다.
