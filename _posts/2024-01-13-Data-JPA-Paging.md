---
title: "[Spring Data JPA] 페이징(Paging)"
date: 2024-01-13 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, paging]
render_with_liquid: false
future: true
---

# **대용량 데이터 손쉽게 처리하기: Spring Data JPA의 페이징 기능**

안녕하세요! 오늘은 Spring Data JPA에서 제공하는 페이징에 대해 더 자세히 알아보겠습니다. 페이징은 대용량 데이터를 처리할 때 필수적인 기능으로, Spring Data JPA를 활용하면 편리하게 구현할 수 있습니다.

## ✅**페이징이란?**

---

우선 페이징이 무엇인지 알아보겠습니다. 페이징은 대량의 데이터를 작은 단위로 나누어 가져오는 기술입니다. 이를 통해 한 번에 모든 데이터를 가져오는 것보다 효율적으로 데이터를 관리할 수 있습니다. 특히 웹 애플리케이션에서는 페이지별로 데이터를 나눠 화면에 보여주는 경우가 많습니다.

## ✅**Spring Data JPA에서의 페이징 구현**

---

Spring Data JPA에서 페이징은 **`Pageable`** 인터페이스를 통해 설정됩니다. 이 인터페이스를 통해 페이지 번호, 페이지 크기, 정렬 조건 등을 설정할 수 있습니다. 그리고 **`Page`** 객체를 통해 페이징된 결과를 받을 수 있습니다.

### **예시 코드**

```java

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // 페이지별 상품 목록 가져오기
    public Page<Product> getProductsByPage(int pageNumber, int pageSize) {
        Pageable pageable = PageRequest.of(pageNumber, pageSize);

        return productRepository.findAll(pageable);
    }
}

```

이렇게 간단한 코드로 페이지별 상품 목록을 가져올 수 있습니다. **`Pageable`**을 생성하고 **`findAll(pageable)`** 메서드를 호출하면, 해당 페이지의 데이터를 가져올 수 있습니다.

## ✅**정렬 조건 추가하기**

---

때로는 특정 기준으로 데이터를 정렬하여 가져오고 싶을 수 있습니다. Spring Data JPA에서는 **`Sort`** 객체를 통해 정렬 조건을 추가할 수 있습니다.

### **예시 코드**

```java

@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    // 정렬된 페이지별 상품 목록 가져오기
    public Page<Product> getSortedProductsByPage(int pageNumber, int pageSize, String sortBy) {
        Pageable pageable = PageRequest.of(pageNumber, pageSize, Sort.by(sortBy));

        return productRepository.findAll(pageable);
    }
}

```

이렇게 **`Sort.by()`** 메서드를 사용하여 정렬 조건을 추가할 수 있습니다.

## ✅**Pageable의 개념**

---

**`Pageable`**은 인터페이스로, 페이지 요청 정보를 캡슐화한 객체입니다. 주로 다음과 같은 메서드가 사용됩니다.

- **`getPageNumber()`**: 현재 페이지 번호를 반환합니다.
- **`getPageSize()`**: 페이지당 아이템 수를 반환합니다.
- **`getSort()`**: 정렬 조건을 반환합니다.

### **예시 코드**

```java

public Page<Product> getProductsByPageAndSort(int pageNumber, int pageSize, String sortBy) {
    Pageable pageable = PageRequest.of(pageNumber, pageSize, Sort.by(sortBy));

    return productRepository.findAll(pageable);
}

```

이렇게 **`PageRequest.of(pageNumber, pageSize, Sort.by(sortBy))`**와 같이 사용하여 페이지와 정렬을 함께 다룰 수 있습니다.

## 📌**주의사항과 팁**

---

- **정렬 조건의 중요성**: 페이징을 사용할 때는 정렬 조건을 명시하는 것이 좋습니다. 그렇지 않으면 데이터의 일관성이 없어질 수 있습니다.
- **인덱스 확인**: 대량의 데이터를 다루는 경우에는 데이터베이스에서 필요한 인덱스가 생성되어 있는지 확인하여 성능을 향상시킬 수 있습니다.
- **큰 데이터셋에서의 효율성**: 대용량의 데이터를 처리할 때는 적절한 페이지 크기를 선택하여 효율성을 고려해야 합니다.
