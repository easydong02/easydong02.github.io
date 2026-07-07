---
title: "[SQL] 시퀀스 조인 외래키2"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

저번엔 `emp`, `dept` 테이블을 그냥 조건문(`where`)으로 연결했다. 이번엔 테이블을 만들 때부터 **외래키(Foreign Key)**로 **시스템적으로 연결**해본다.

> **목표**: `category`(부모)와 `subject`(자식) 테이블을 외래키로 연결. 각 테이블은 primary key를 가진다.

```
category (부모)              subject (자식)
 category_id (PK) ◄──────── category_id (FK)
 category_name               subject_id (PK)
                             subject_name, target
```

---

## 1. 부모 테이블 — category

```sql
create table category(
    category_id number primary key,   -- 이 PK를 자식이 참조
    category_name varchar(25)
);
```

---

## 2. 자식 테이블 — subject (외래키 지정)

```sql
create table subject(
    subject_id number primary key,
    category_id number,
    subject_name varchar(15),
    target varchar(8),
    constraint fk_category_subject
        foreign key(category_id) references category(category_id)
);
```

마지막 줄이 핵심이다. 외래키 선언을 뜯어보면:

| 구성 | 의미 |
|------|------|
| `constraint fk_category_subject` | 제약조건 이름 (fk = foreign key) |
| `foreign key(category_id)` | subject의 `category_id`를 외래키로 |
| `references category(category_id)` | 부모 `category`의 PK를 참조 |

> 💡 이제 `subject.category_id`에는 **부모 `category`에 존재하는 값만** 넣을 수 있다(참조 무결성). 없는 카테고리를 참조하면 에러가 난다. 이렇게 두 테이블이 **시스템적으로 연결**된다.

---

## 📝 정리

```
외래키로 테이블 연결
├─ 부모   PK를 가진 테이블 (category)
├─ 자식   FK로 부모의 PK를 참조 (subject)
└─ 문법   constraint 이름 foreign key(열) references 부모(PK)
```

| 개념 | 한 줄 정의 |
|------|------|
| **외래키(FK)** | 다른 테이블의 PK를 참조하는 컬럼 |
| **참조 무결성** | 부모에 존재하는 값만 자식에 입력 가능 |

지난번의 `where` 조인이 "그때그때 연결"이라면, 외래키는 **테이블 설계 단계에서 관계를 못 박는 것**이다. 참조 무결성 덕분에 잘못된 데이터가 들어오는 것을 원천 차단할 수 있다.
