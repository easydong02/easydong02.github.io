---
title: "[Spring Data JPA] 쿼리 메소드"
date: 2024-01-12 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, update, delete]
render_with_liquid: false
future: true
---

# ✅**Spring Data JPA에서 데이터 수정과 삭제: 쉽고 간편하게!**

---

안녕하세요! 오늘은 Spring Data JPA에서 데이터를 수정하고 삭제하는 방법에 대해 알아보겠습니다. Spring Data JPA는 객체 관계 매핑(ORM)을 통해 데이터베이스 조작을 편리하게 해주는 도구로, CRUD(Create, Read, Update, Delete) 기능을 간단하게 처리할 수 있습니다.

## ✅**데이터 수정하기**

---

Spring Data JPA에서 데이터를 수정하는 과정은 간단합니다. 엔티티를 조회하여 원하는 필드를 수정하고, 이를 저장하면 됩니다.

### **예시 코드**

```java

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // 상품 정보 수정
    public Product updateProduct(Long productId, String newProductName) {
        // 데이터베이스에서 엔티티 조회
        Optional<Product> optionalProduct = productRepository.findById(productId);

        // 엔티티가 존재하는지 확인 후 수정
        if (optionalProduct.isPresent()) {
            Product product = optionalProduct.get();
            product.setProductName(newProductName);

            // 수정된 엔티티 저장
            return productRepository.save(product);
        }

        // 수정할 상품이 없는 경우
        return null;
    }
}

```

위 코드에서 **`ProductService`** 클래스는 상품 정보를 수정하는 서비스입니다. **`productRepository.findById()`**를 통해 엔티티를 조회하고, 필요한 필드를 수정한 후 **`productRepository.save()`**로 저장합니다.

## ✅**데이터 삭제하기**

---

Spring Data JPA를 사용하면 간단하게 데이터를 삭제할 수 있습니다. 삭제할 데이터의 ID를 이용하여 삭제하는 메서드를 작성합니다.

### **예시 코드**

```java

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // 상품 정보 삭제
    public void deleteProduct(Long productId) {
        // 데이터베이스에서 엔티티 조회
        Optional<Product> optionalProduct = productRepository.findById(productId);

        // 엔티티가 존재하는지 확인 후 삭제
        optionalProduct.ifPresent(product -> productRepository.delete(product));
    }
}

```

여기서 **`productRepository.delete(product)`** 메서드를 호출하면 해당 엔티티가 삭제됩니다. 만약 엔티티가 존재하지 않으면 무시됩니다.

## 📌**주의사항과 팁**

---

- **Cascade 설정**: 연관 관계에서 삭제 시 연쇄적으로 삭제하고 싶다면 엔티티 간의 연관 관계에서 **`CascadeType.REMOVE`** 등을 고려해보세요.
- **Optimistic Locking**: 동시에 여러 사용자가 동일한 데이터를 수정하려는 경우를 대비하여 낙관적 락킹(Optimistic Locking) 등을 사용할 수 있습니다.
- **삭제 시 참조 무결성**: 삭제 시 연관된 데이터의 무결성을 유지하기 위해 관련된 데이터를 먼저 삭제하거나 적절한 전략을 고려해야 합니다.
