---
title: "[Spring Data JPA] 쿼리 메소드"
date: 2024-01-12 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, update, delete]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 Spring Data JPA로 데이터를 **수정(Update)**하고 **삭제(Delete)**하는 방법을 정리한다.

> Spring Data JPA는 ORM으로 DB 조작을 편리하게 해주며, CRUD를 간단히 처리할 수 있다.

---

## 1. 데이터 수정 (Update)

수정은 **조회 → 필드 변경 → 저장**의 흐름이다.

```java
@Service
public class ProductService {
    @Autowired
    private ProductRepository productRepository;

    public Product updateProduct(Long productId, String newProductName) {
        Optional<Product> optionalProduct = productRepository.findById(productId);  // 조회

        if (optionalProduct.isPresent()) {
            Product product = optionalProduct.get();
            product.setProductName(newProductName);          // 필드 수정
            return productRepository.save(product);          // 저장
        }
        return null;   // 없으면 null
    }
}
```

> 💡 사실 `@Transactional` 안에서는 조회한 엔티티의 필드만 바꿔도 **변경 감지(dirty checking)**로 자동 업데이트된다. `save()`는 명시적 저장이다.

---

## 2. 데이터 삭제 (Delete)

```java
public void deleteProduct(Long productId) {
    Optional<Product> optionalProduct = productRepository.findById(productId);
    optionalProduct.ifPresent(product -> productRepository.delete(product));
}
```

> 💡 `ifPresent`로 **엔티티가 존재할 때만** 삭제한다. 없으면 조용히 무시된다.

| 작업 | 메소드 |
|------|------|
| 수정 | `findById` → `setXxx` → `save` |
| 삭제 | `findById` → `delete` |

---

## 📌 주의사항과 팁

| 항목 | 내용 |
|------|------|
| **Cascade 설정** | 연관 관계 삭제를 연쇄하려면 `CascadeType.REMOVE` 고려 |
| **낙관적 락(Optimistic Locking)** | 동시 수정 대비 |
| **참조 무결성** | 삭제 시 연관 데이터를 먼저 처리 |

---

## 📝 정리

```
수정 & 삭제
├─ 수정   조회 → 필드 변경 → save (또는 변경 감지)
├─ 삭제   조회 → ifPresent → delete
└─ 주의   Cascade / 낙관적 락 / 참조 무결성
```

| 개념 | 한 줄 정의 |
|------|------|
| **변경 감지** | 트랜잭션 내 필드 변경 자동 반영 |
| **ifPresent** | 존재할 때만 처리 |

수정과 삭제 모두 **먼저 조회한 뒤** 처리하는 것이 안전하다. 특히 연관 관계가 있다면 Cascade와 참조 무결성을 함께 고려해야 예기치 않은 오류를 막을 수 있다.
