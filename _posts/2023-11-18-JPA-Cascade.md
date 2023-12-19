---
title: "[JPA] 영속성 전이(Cascade)"
date: 2023-11-18 00:00:00 +0900
categories: [Database, JPA]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

# 영속성 전이(Cascade)

## 영속성 전이란?

---

- 특정 엔티티를 영속 상태롤 만들 때 연관된 엔티티도 같이 영속성 상태로 만들고 싶을 때
- ex) 부모 저장할 때 자식도 같이 저장하고 싶을 때

Parent

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> childList = new ArrayList<>();
```

Child

```java
@ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
```

```java
Child child = new Child();
            Child child2 = new Child();

            Parent parent = new Parent();

            parent.addChild(child);
            parent.addChild(child2);

            em.persist(parent);

---

Hibernate:
    /* insert hellojpa.Parent
        */ insert
        into
            Parent
            (name, parent_id)
        values
            (?, ?)
Hibernate:
    /* insert hellojpa.Child
        */ insert
        into
            Child
            (name, parent_id, id)
        values
            (?, ?, ?)
Hibernate:
    /* insert hellojpa.Child
        */ insert
        into
            Child
            (name, parent_id, id)
        values
            (?, ?, ?)
```

## 영속성 전이: CASCADE - 주의!

---

- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련 없다!
- 그저 연관된 엔티티도 함께 영속화하는 편리함만 제공한다.
- cascade는 종속되는 객체 둘다 라이프 사이클이 같을 때 사용하는게 좋다.
- 그리고 위 예시에서 Child 객체가 다른 객체랑 엮이지 않을 때 사용하는게 좋다.
