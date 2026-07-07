---
title: "[Spring Data JPA] 페이징(Paging)"
date: 2024-01-13 00:00:00 +0900
categories: [Backend, Spring-Data-JPA]
tags: [springdatajpa, paging]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 대용량 데이터를 다룰 때 필수인 **페이징(Paging)**을 Spring Data JPA로 구현하는 법을 정리한다.

> **페이징이란?**
> 대량의 데이터를 **작은 단위로 나누어** 가져오는 기술. 한 번에 모든 데이터를 가져오는 것보다 효율적이며, 웹에서 페이지별로 보여줄 때 자주 쓰인다.

---

## 1. Pageable과 Page

Spring Data JPA의 페이징은 **`Pageable`**(요청 정보)과 **`Page`**(결과)로 이뤄진다.

```java
public Page<Product> getProductsByPage(int pageNumber, int pageSize) {
    Pageable pageable = PageRequest.of(pageNumber, pageSize);   // 요청 정보 생성
    return productRepository.findAll(pageable);                 // Page 반환
}
```

| 요소 | 역할 |
|------|------|
| `PageRequest.of(번호, 크기)` | 페이지 요청 생성 |
| `findAll(pageable)` | 해당 페이지 데이터 조회 |
| `Page<T>` | 페이징된 결과 (전체 개수·총 페이지 포함) |

---

## 2. 정렬 조건 추가 (Sort)

```java
public Page<Product> getSortedProductsByPage(int pageNumber, int pageSize, String sortBy) {
    Pageable pageable = PageRequest.of(pageNumber, pageSize, Sort.by(sortBy));
    return productRepository.findAll(pageable);
}
```

**`Sort.by(정렬기준)`**을 `PageRequest`에 넣어 페이징과 정렬을 함께 처리한다.

---

## 3. Pageable의 주요 메소드

```java
Pageable pageable = PageRequest.of(pageNumber, pageSize, Sort.by(sortBy));
```

| 메소드 | 반환 |
|------|------|
| `getPageNumber()` | 현재 페이지 번호 |
| `getPageSize()` | 페이지당 아이템 수 |
| `getSort()` | 정렬 조건 |

---

## 📌 주의사항과 팁

| 항목 | 내용 |
|------|------|
| **정렬 조건 명시** | 없으면 데이터 일관성이 깨질 수 있음 |
| **인덱스 확인** | 대량 데이터는 인덱스로 성능 향상 |
| **페이지 크기** | 대용량은 적절한 크기 선택으로 효율 확보 |

---

## 📝 정리

```
페이징
├─ 요청    PageRequest.of(번호, 크기 [, Sort])
├─ 조회    findAll(pageable) → Page<T>
├─ 정렬    Sort.by(기준)
└─ 주의    정렬 명시 + 인덱스 + 페이지 크기
```

| 개념 | 한 줄 정의 |
|------|------|
| **Pageable** | 페이지 요청 정보(번호·크기·정렬) |
| **Page** | 페이징 결과(데이터 + 메타) |
| **Sort** | 정렬 조건 |

페이징은 대용량 데이터를 **나눠서 효율적으로** 다루는 필수 기능이다. `PageRequest.of(...)` 하나로 페이지·크기·정렬을 지정할 수 있어, 몇 줄로 견고한 페이징을 구현할 수 있다.
