---
title: "[JPA] 고아객체"
date: 2023-11-19 00:00:00 +0900
categories: [Backend, JPA]
tags: [jpa]
render_with_liquid: false
future: true
---

## 📌 들어가며

부모와의 관계가 끊어진 자식 엔티티를 **고아 객체(orphan)**라 한다. JPA는 이 고아 객체를 자동으로 DB에서 삭제하는 기능을 제공한다.

> **고아 객체**: 부모 엔티티와 **관계가 끊어진** 자식 엔티티.

---

## 1. orphanRemoval

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Child> childList = new ArrayList<>();
```

`orphanRemoval = true`를 주면, **컬렉션에서 자식을 제거하면 DB에서도 DELETE**된다.

```java
Parent findParent = em.find(Parent.class, parent.getId());
findParent.getChildList().remove(0);   // 컬렉션에서 제거
```

```sql
-- 자동으로 delete!
delete from Child where id=?
```

---

## 2. ⚠️ 주의사항

> ⚠️ **참조가 끊어진 것을 아예 DB에서 지워버리므로** 매우 조심해야 한다.

| 규칙 | 설명 |
|------|------|
| **단일 참조일 때만** | 자식을 참조하는 곳이 **하나**일 때 사용 |
| **부모 제거 = 자식 제거** | 부모를 지우면 자식도 삭제 (`CascadeType.REMOVE`와 같은 효과) |
| **적용 대상** | `@OneToOne`, `@OneToMany`에서만 가능 |

```java
findParent.getChildList().clear();   // 전체 제거 → 모든 자식 delete
```

---

## 3. Cascade.ALL + orphanRemoval = 생명주기 위임

> 💡 **`CascadeType.ALL` + `orphanRemoval = true`**를 함께 쓰면, **자식 객체의 생명주기를 부모가 완전히 관리**하게 된다. 부모를 통해 자식을 생성·삭제할 수 있어, **도메인 주도 설계(DDD)** 구현에 유용하다.

```
CascadeType.ALL  ─┐
                  ├─► 자식의 생명주기를 부모가 관리 (DDD)
orphanRemoval=true ┘
```

---

## 📝 정리

```
고아 객체(orphanRemoval)
├─ 정의   부모와 관계 끊긴 자식
├─ 동작   컬렉션에서 제거 → DB DELETE
├─ 주의   단일 참조일 때만! 부모 제거=자식 제거
└─ 조합   Cascade.ALL + orphanRemoval → 생명주기 위임(DDD)
```

| 개념 | 한 줄 정의 |
|------|------|
| **고아 객체** | 부모 참조가 끊긴 자식 |
| **orphanRemoval** | 참조 끊기면 자동 삭제 |
| **생명주기 위임** | 부모가 자식의 생성·삭제 관리 |

`orphanRemoval`은 강력하지만 위험하다. **자식이 오직 한 부모에게만 속할 때** 써야 하며, Cascade.ALL과 결합하면 부모가 자식의 생명주기를 온전히 책임지는 DDD 스타일 설계가 가능해진다.
