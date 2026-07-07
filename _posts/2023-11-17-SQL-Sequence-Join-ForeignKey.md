---
title: "[SQL] 시퀀스 조인 외래키"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 자동 증가를 담당하는 **시퀀스(Sequence)**, 두 테이블을 잇는 **조인(Join)**, 그 축이 되는 **외래키(Foreign Key)**를 정리한다.

---

## 1. 시퀀스 (Sequence)

> MySQL에는 `auto_increment`가 있지만 **오라클에는 없다.** 그래서 자동 증가를 위해 **시퀀스**를 따로 만들어야 한다.

먼저 권한이 필요하다. (관리자 `system`으로 접속 후)

```sql
grant create sequence to batman;   -- batman에게 시퀀스 생성 권한 부여
```

이제 `batman`으로 시퀀스를 만든다.

```sql
create sequence seq
    increment by 1     -- 1씩 증가
    start with 1;      -- 1부터 시작
```

**테이블에서 사용** — `시퀀스.nextval`로 다음 값을 얻는다.

```sql
create table family(
    family_id number primary key,   -- 오라클은 int 대신 number
    name varchar(25),
    age number,
    job varchar(25)
);

insert into family(family_id, name, age, job)
    values(seq.nextval, 'spiderman', 78, 'police');   -- family_id에 자동 증가값
```

> 💡 `seq.nextval`은 마지막 순번 뒤에 차례로 값을 넣는다. 레코드가 없으면 마지막이 0이니 **1**이 들어간다. 시퀀스를 잘 활용하면 1씩 증가 외에 나만의 방식으로도 자동 증가를 만들 수 있다.

---

## 2. 조인 (Join)

> **조인**은 두 테이블을 이어주는 작업이다. **외래키를 축**으로 연결한다.

`emp`(직원)와 `dept`(부서)를 그냥 함께 부르면?

```sql
select * from emp, dept;   -- 56 rows! (14 × 4)
```

![Desktop View](/assets/img/Database/SQL/Seq-Join-ForeignKey/2.png)

> ⚠️ `emp` 14개 × `dept` 4개 = **56개**가 서로 중복 연결된다(카티션 곱). 우리가 원하는 건 이게 아니다.

**deptno가 같은 것끼리만** 연결한다.

```sql
select * from emp, dept where emp.deptno = dept.deptno;   -- 14 rows
```

![Desktop View](/assets/img/Database/SQL/Seq-Join-ForeignKey/3.png)

```
emp.deptno ─┐
            ├─ 같은 deptno끼리 짝지음 → 14개 (올바른 조인)
dept.deptno ┘
```

> 💡 두 테이블을 잇는 **외래키가 `deptno`**다. `where`로 "양쪽의 deptno가 같은 것만" 조건을 걸어 올바르게 연결한다.

---

## 📝 정리

```
시퀀스 · 조인 · 외래키
├─ 시퀀스   오라클의 자동증가 (create sequence, seq.nextval)
├─ 조인     두 테이블 연결 (그냥 부르면 카티션 곱!)
└─ 조건조인  where 테이블A.키 = 테이블B.키
```

| 개념 | 한 줄 정의 |
|------|------|
| **시퀀스** | 오라클의 자동 증가 객체 |
| **nextval** | 시퀀스의 다음 값 |
| **조인** | 외래키를 축으로 두 테이블 연결 |
| **카티션 곱** | 조건 없이 조인 시 모든 조합(14×4) |

오라클은 자동 증가를 **시퀀스**로 처리하고, 정규화된 테이블은 **외래키를 축으로 조인**해 다시 합친다. 조인 시 조건(`where`)이 없으면 카티션 곱이 발생한다는 점을 꼭 기억하자.
