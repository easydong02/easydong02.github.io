---
title: "[JPA] 고아객체"
date: 2023-11-19 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa]
render_with_liquid: false
future: true
---

# 고아 객체

## 고아 객체?

---

- 부모 엔티티와 관계가 끊어진 자식 엔티티

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Child> childList = new ArrayList<>();
```

- orphanRemoval를 true로 주면 저 객체가 사라지면 db에서도 delete됨.

```java
Child child = new Child();
Child child2 = new Child();

Parent parent = new Parent();

parent.addChild(child);
parent.addChild(child2);

em.persist(parent);

em.flush();
em.clear();

Parent findParent = em.find(Parent.class, parent.getId());
findParent.getChildList().remove(0);

---

/* delete hellojpa.Child */ delete
        from
            Child
        where
            id=?
```

## 고아 객체 - 주의

---

- 참조가 되지 않는 것은 아예 db에서 지워버리기 때문에 조심해야 함.
- 참조하는 곳이 하나일 때 사용해야 겠죠?
- 부모를 제거하면 자식도 제거됨( = CascadeType.REMOVE, ALL)

```java
Parent findParent = em.find(Parent.class, parent.getId());
findParent.getChildList().clear();

---

Hibernate:
    /* delete hellojpa.Child */ delete
        from
            Child
        where
            id=?
Hibernate:
    /* delete hellojpa.Child */ delete
        from
            Child
        where
            id=?
```

- OneToOne, ManyToOne에서만 가능!

## 영속성 전이 + 고아 객체, 생명주기

---

- CascadeType.ALL 과 orphanRemoval = true를 같이쓰면? → **자식 객체의 생명 주기를 부모 객체에서 관리가 가능하다**!
- **도메인 주도 설계(DDD)** 구현할 때 유용!
