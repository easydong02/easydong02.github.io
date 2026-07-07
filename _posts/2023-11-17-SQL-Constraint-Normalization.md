---
title: "[SQL] 제약조건 정규화"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 테이블을 만들 때의 **제약조건(constraint)**과 **정규화(normalization)**를 정리한다.

> **왜 제약조건이 필요한가?** `emp` 테이블에서 이름·직급은 중복될 수 있다. 만약 모든 값이 똑같은 사람 둘이 들어오면, 하나만 지우려 할 때 **구분할 방법이 없다.** 그래서 **고유값을 가지는 필드**를 정하고, 테이블 생성 시부터 제약을 건다.

![Desktop View](/assets/img/Database/SQL/Constraint-Normalization/1.png)

---

## 1. 제약조건 (constraint)

> ⚠️ 제약조건은 **테이블을 생성할 때만** 지정할 수 있으므로 신중히 설계해야 한다.

| 제약조건 | 역할 |
|------|------|
| **Unique** | 각 값이 **고유**(중복 insert 시 에러) |
| **Not NULL** | 값을 **반드시** 입력해야 함 |
| **Primary Key** | Unique + Not NULL (고유 식별) |
| **Auto_increment** | 값을 자동으로 1씩 증가 |

### Unique + Not NULL

`Unique`만 걸면 값을 안 넣어 **NULL**이 될 수 있어, 완전히 똑같은 레코드 둘이 들어올 수 있다. 그래서 **`Not NULL`을 함께** 건다.

![Desktop View](/assets/img/Database/SQL/Constraint-Normalization/3.png)

> 💡 이제 `member_id`는 **중복도 불가, 값 생략도 불가** → 항상 고유값이 생긴다.

### Primary Key + Auto_increment

`Primary Key`는 Unique와 Not NULL을 합친 것이다. 여기에 **`Auto_increment`**를 더하면 값을 직접 안 적어도 **1부터 자동으로 증가**한다.

```sql
member_id int primary key auto_increment
```

> 💡 이러면 **고유값이면서 빠짐없이 순차 증가**하는 완벽한 식별자가 된다. (3 다음에 5를 넣는 실수도 방지)

---

## 2. 정규화 (Normalization)

![Desktop View](/assets/img/Database/SQL/Constraint-Normalization/4.png)

> 쇼핑몰 DB를 만든다고 하자. 상품마다 카테고리(상의·하의 등)를 매번 반복해서 적으면 **중복이 엄청 많아진다.** 프로그래밍의 목적은 결국 **중복을 최소화하고 효율적으로 관리**하는 것이다.

**정규화**란 이런 **중복되는 카테고리를 하나로 묶어 별도 테이블로 분리**하는 것이다.

```
[정규화 전]                    [정규화 후]
상품 테이블                    상품 테이블 ──┐
 상품A | 상의 | ...             상품A | cat_id=1     │  카테고리 테이블
 상품B | 상의 | ...     →       상품B | cat_id=1     └► 1 | 상의
 상품C | 하의 | ...             상품C | cat_id=2        2 | 하의
 (중복 다수)                   (중복 제거)
```

> 💡 정규화는 특정 기술이라기보다, **중복을 줄이려는 개발자의 사고방식**에 가깝다.

---

## 📝 정리

```
제약조건 & 정규화
├─ Unique        고유값 (중복 불가)
├─ Not NULL      값 필수
├─ Primary Key   Unique + Not NULL
├─ Auto_increment  자동 증가
└─ 정규화        중복 카테고리를 별도 테이블로 분리
```

| 개념 | 한 줄 정의 |
|------|------|
| **Primary Key** | 고유 식별 컬럼 (Unique+Not NULL) |
| **Auto_increment** | 값을 자동으로 순차 증가 |
| **정규화** | 중복 제거를 위한 테이블 분리 |

제약조건은 **데이터의 무결성**을, 정규화는 **중복 최소화**를 담당한다. 특히 `primary key auto_increment`는 실무에서 가장 흔히 쓰는 식별자 패턴이니 꼭 기억하자.
