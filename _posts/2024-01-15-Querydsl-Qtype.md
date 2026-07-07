---
title: "[Querydsl] Q-타입(Q-type)"
date: 2024-01-15 00:00:00 +0900
categories: [Backend, Querydsl]
tags: [querydsl, qtype]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 QueryDSL의 핵심인 **Q-Type**을 정리한다.

> **QueryDSL이란?** 자바 기반의 오픈소스 SQL 프로젝션 라이브러리로 JPA·Hibernate를 지원한다.
> **Q-Type이란?** 엔티티의 필드를 **정적 상수로 선언한 클래스**. 쿼리 작성 시 **타입 안전성**을 제공한다.

---

## 1. Q-Type 사용하기

`Product` 엔티티가 있다고 하자.

```java
@Entity
public class Product {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private int price;
    private String category;
}
```

자동 생성된 **`QProduct.product`**로 타입 안전한 쿼리를 작성한다.

```java
import static com.example.model.QProduct.product;

public List<Product> findProductsInCategory(String category) {
    return new JPAQueryFactory(entityManager)
        .selectFrom(product)
        .where(product.category.eq(category))   // 필드 접근 시 오타 방지
        .fetch();
}
```

> 💡 `product.category.eq(category)`처럼 필드에 접근하면, **문자열 JPQL과 달리 컴파일 시점에 오타를 잡아준다.** 이것이 QueryDSL의 가장 큰 장점인 **타입 안전성**이다.

| 방식 | 오타 검출 |
|------|------|
| 문자열 JPQL | 런타임에 발견 |
| **Q-Type** | **컴파일 시점**에 발견 |

---

## 📌 주의사항과 팁

| 항목 | 내용 |
|------|------|
| **빌드 시 생성 확인** | Q-Type은 코드 생성으로 만들어짐 → 빌드 후 확인 |
| **엔티티와 일치** | Q-Type 필드는 엔티티 필드와 일치해야 함 |
| **자동 완성 활용** | IDE 자동완성으로 빠르고 정확하게 |

---

## 📝 정리

```
QueryDSL Q-Type
├─ 정체    엔티티 필드를 상수로 선언한 클래스
├─ 사용    QProduct.product.category.eq(...)
├─ 장점    타입 안전성 (컴파일 시 오타 검출)
└─ 생성    빌드 시 자동 생성
```

| 개념 | 한 줄 정의 |
|------|------|
| **QueryDSL** | 타입 안전한 쿼리 라이브러리 |
| **Q-Type** | 엔티티 필드를 상수화한 클래스 |
| **타입 안전성** | 컴파일 시점 오타 검출 |

Q-Type은 QueryDSL의 심장이다. 문자열 쿼리의 오타 위험을 **컴파일 시점에 제거**해주니, 복잡한 동적 쿼리도 안전하게 작성할 수 있다.
