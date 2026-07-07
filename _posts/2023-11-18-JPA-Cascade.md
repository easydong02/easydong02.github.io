---
title: "[JPA] 영속성 전이(Cascade)"
date: 2023-11-18 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa, cascade]
render_with_liquid: false
future: true
---

## 📌 들어가며

부모 엔티티를 저장할 때 연관된 자식도 **함께 저장**하고 싶을 때가 있다. 이걸 자동으로 처리해주는 것이 **영속성 전이(Cascade)**다.

> **영속성 전이란?**
> 특정 엔티티를 영속 상태로 만들 때, **연관된 엔티티도 함께 영속화**하는 기능. (예: 부모 저장 시 자식도 함께 저장)

---

## 1. Cascade 설정

```java
// Parent
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
private List<Child> childList = new ArrayList<>();

// Child
@ManyToOne
@JoinColumn(name = "parent_id")
private Parent parent;
```

`cascade = CascadeType.ALL`을 걸면, **부모만 `persist`해도 자식이 함께 저장**된다.

```java
Parent parent = new Parent();
parent.addChild(child);
parent.addChild(child2);

em.persist(parent);   // 부모만 persist
```

```sql
-- 자식까지 자동으로 insert!
insert into Parent (name, parent_id) values (?, ?)
insert into Child  (name, parent_id, id) values (?, ?, ?)
insert into Child  (name, parent_id, id) values (?, ?, ?)
```

---

## 2. ⚠️ Cascade 주의사항

> ⚠️ **Cascade는 연관관계 매핑과 아무 관련이 없다.** 그저 연관된 엔티티를 **함께 영속화하는 편리함**만 제공한다.

**언제 쓰면 좋은가:**

| 조건 | 이유 |
|------|------|
| **라이프사이클이 같을 때** | 부모·자식이 함께 생성·삭제될 때 |
| **자식이 단독 소유일 때** | Child가 다른 객체와 엮이지 않을 때 |

> 💡 자식이 여러 부모에서 참조되는 상황이라면 Cascade는 위험하다. **한 부모가 자식의 생명주기를 온전히 관리할 때**만 쓰자.

---

## 📝 정리

```
영속성 전이(Cascade)
├─ 기능   부모 영속화 시 자식도 함께 영속화
├─ 설정   @OneToMany(cascade = CascadeType.ALL)
├─ 주의   연관관계 매핑과 무관, 편의 기능일 뿐
└─ 조건   라이프사이클 동일 + 자식 단독 소유일 때
```

| 개념 | 한 줄 정의 |
|------|------|
| **Cascade** | 연관 엔티티를 함께 영속화 |
| **CascadeType.ALL** | 모든 영속 작업을 전이 |

Cascade는 "부모를 저장하면 자식도 저장"이라는 편의 기능이다. 다만 **자식이 그 부모에게만 속할 때** 써야 안전하다. 여러 곳에서 참조되는 자식에 Cascade를 걸면 예기치 않은 부작용이 생긴다.
