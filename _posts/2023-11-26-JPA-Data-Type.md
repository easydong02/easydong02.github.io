---
title: "[JPA] 값 타입"
date: 2023-11-26 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa, hibernate]
render_with_liquid: false
future: true
---

# 값 타입

## 엔티티 타입

---

- @Entity로 정의합니다.
- 데이터가 변해도 식별자로 지속해서 **추적 가능**.

## 값 타입 분류

---

### 기본값 타입

- 자바 기본 타입(int, double)
- 래퍼 클래스(Integer, Long)
- String

### 임베디드 타입(embedded type, 복합 값 타입)

### 컬렉션 값 타입(collection value type)

## 기본값 타입

---

- int, long, String 같은 단순히 값으로 사용하는 자바 기본 타입
- 식별자가 없으므로 값 변경시 **추적 불가**
- 생명주기가 엔티티에 의존
- 값 타입은 공유하면 안 된다.(사이드 이펙트 x)
- 값 복사(call by value)

## 임베디드 타입

---

- 새로운 값 타입을 사용자 정의로 생성 가능.
- Member 타입처럼 주로 기본값 타입을 모아서 만든 복합 타입
- int, String과 같은 값 타입(추적 안 돼서 변경하면 끝)
- 여러 엔티티에서 공유하면 위험함!! 왜냐하면 임베디드 값이 바뀌면 종속하는 모든 엔티티의 값이 바뀌기 때문! 따라서 같은 값을 넣고 싶으면 새로운 임베디드 값을 만들고 밸류 복사하는게 좋다!

### 임베디드 타입 사용법

---

- @Embeddable: 값 타입을 정의한느 곳에 표시
- @Embedded: 값 타입을 사용하는 곳에 표시
- 기본 생성자는 필수!

### 임베디드 타입의 장점

---

- 재사용 가능!
- 높은 응집도 구현
- Period.isWork()처럼 해당 값 타입만 사용하는 의미 있는 메소드 만들 수 있음
- 임베디드가 소유되어 있는 엔티티에 생명주기를 의존함

### 임베디드 타입과 테이블 매핑

---

- 임베디드 타입도 결국 엔티티의 값일 뿐이다
- 임베디드 타입을 사용하기 전과 후의 **매핑하는 테이블은 같다**
- 세밀하게 엔티티 매핑하는 것이 가능
- 잘 설계한 orm 어플은 매핑한 테이블보다 클래스의 수가 더 많

### @AttributeOverride: 속성 재정의

---

- 한 엔티티에서 같은 값 타입을 사용하기 위해 사용
- 컬럼 명이 중복 됨

### **불변 객체**

---

- 객체 타입의 사고를 막으려면 **원천 차단해야함.**
- 생성자로만 값을 설정하고 **Setter를 만들지 않으면 됨.**

## 값 타입 비교

---

- 동일성 비교: 인스턴스의 참조 값을 비교(==)
- 동등성 비교: 인스턴스 값 비교 (equals())
- 그래서 되도록 equals()를 쓰고 이것을 재정의 하는 것이 좋다.(이때 getter를 쓰는게 좋다. 왜냐하면 그냥 필드로 접근하면 프록시 객체일 때 접근을 못하기 때문)

```java
@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getCity(), getStreet(), getZipcode());
    }
```

## 값 타입 컬렉션

---

- 값 타입을 하나 이상 저장할 때 사용한다.
- @ElementCollection, @CollectionTable 어노테이션 사용한다.
- DB는 컬렉션을 같은 테이블에 저장할 수 없다.
- 따라서 별도의 테이블 필요함.
- 컬렉션은 정의된 엔티티에 의해서 라이프 사이클이 정해진다.
- 따라서 영속성 전이(cascade) + 고아 객체 제거 기능을 같이 있는 거라고 보면 된다.
- 사용할 때는 지연로딩이라 그 컬렉션을 조회 등, 사용할 때 엔티티와 연결한다.
- 수정할 때는 새로운 객체로 갈아 끼워야 한다!

```java
Member findMember = em.find(Member.class, member.getId());
Address address = findMember.getAddress();
findMember.setAddress(new Address("newCity",address.getStreet(), address.getZipcode()));

---
findMember.getAddress().setCity("newCity") <- 이거는 사용하면 안 된다!
```

### **값 타입 컬렉션의 제약사항**

---

- 엔티티와 다르게 식별자 개념이 없어서 값을 변경하면 추적이 어렵다
- **값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.**

```java
//주소이력 바꾸기 equals와 hashCode를 재정의 했기 때문에 new 해도 비교가 가능!
findMember.getAddressHistory().remove(new Address("old1","street","1000"));
findMember.getAddressHistory().add(new Address("newCity1","street","1000"));

---
//한개 삭제하고 한개 저장했는데 다 삭제하고 다시 2개를 넣었다.

Hibernate:
    /* delete collection hellojpa.Member.addressHistory */ delete
        from
            ADDRESS
        where
            MEMBER_ID=?
Hibernate:
    /* insert collection
        row hellojpa.Member.addressHistory */ insert
        into
            ADDRESS
            (MEMBER_ID, city, street, zipcode)
        values
            (?, ?, ?, ?)
Hibernate:
    /* insert collection
        row hellojpa.Member.addressHistory */ insert
        into
            ADDRESS
            (MEMBER_ID, city, street, zipcode)
        values
            (?, ?, ?, ?)
```

- 값 타입 컬렉션의 기본키는 모든 컬럼 묶어서 구성: **null 안되고, 중복 x**

### 값 타입 컬렉션 대안

---

- 그냥 **실무에서는 일대다 관계**를 고려

**`AddressEntity.java`**

```java
@Id
    @GeneratedValue
    private Long id;
    private Address address;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
```

**`Member.java`**

```java
@OneToMany(mappedBy = "member",cascade = CascadeType.ALL, orphanRemoval = true)
private List<AddressEntity> addressEntityList = new ArrayList<>();
```

`**main.java**`

```java
AddressEntity addressEntity1= new AddressEntity("old1","street","10000");
            AddressEntity addressEntity2= new AddressEntity("old2","street","10000");

            addressEntity1.setMember(member);
            addressEntity2.setMember(member);

            member.getAddressEntityList().add(addressEntity1);
            member.getAddressEntityList().add(addressEntity2);

            em.persist(member);
```
