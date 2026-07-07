---
title: "[SQL] 테이블 생성 삽입 조회"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

데이터베이스 시간이다! 이번 글에서는 MySQL로 **DB·테이블 생성 → 데이터 삽입 → 조회**의 기본 흐름을 익힌다.

> **데이터베이스란?** 엑셀처럼 정보들의 유의미한 집합을 종류별로 나누어 저장하는 **창고** 같은 것.

```
DBMS
 └─ 데이터베이스 (myDB)
      └─ 테이블 (heros, emp, dept)
           └─ 레코드(행) / 컬럼(열)
```

---

## 1. DB 다루기 기본 명령어

| 명령 | 역할 |
|------|------|
| `show databases;` | DB 목록 조회 |
| `create database myDB;` | DB 생성 |
| `use myDB;` | 해당 DB 선택 |

![Desktop View](/assets/img/Database/SQL/Create-Table/3.png)
![Desktop View](/assets/img/Database/SQL/Create-Table/4.png)

> 💡 테이블을 만들기 전에 `use`로 DB를 **선택**해야 그 안에 만들어진다. (이클립스의 패키지-클래스 관계와 비슷)

---

## 2. 테이블 생성과 삽입

```sql
create table heros(
    name    char(20),
    age     int,
    ability char(20)
) default character set utf8;   -- 한글 위해 utf8
```

| 요소 | 의미 |
|------|------|
| `char(20)` | 문자 타입, (20)은 용량 |
| `int` | 정수 타입 |
| `default character set utf8` | 한글 사용을 위한 인코딩 |

```sql
insert into heros(name, age, ability) values('배트맨', 32, '날기');
select name, age, ability from heros;
```

![Desktop View](/assets/img/Database/SQL/Create-Table/5.png)

---

## 3. 연습용 DB (dept · emp)

오라클이 학습용으로 배포한 **부서(dept)·직원(emp)** 테이블을 만든다.

```sql
create table dept(
    deptno int primary key auto_increment,   -- 기본키 + 자동 증가
    dname  varchar(14),
    loc    varchar(13)
);
insert into dept values(10, 'ACCOUNTING', 'NEW YORK');
-- ... RESEARCH, SALES, OPERATIONS

create table emp(
    empno    int primary key auto_increment,
    ename    varchar(10),
    job      varchar(9),
    mgr      int,
    hiredate timestamp,
    sal      int,
    comm     int,
    deptno   int
);
insert into emp(empno,ename,job,mgr,hiredate,sal,deptno)
    values(7369,'SMITH','CLERK',7902,'80/12/17',800,20);
-- ... (14명의 직원)
```

> 💡 `primary key auto_increment`는 **기본키(중복 불가·NOT NULL)**이면서 값을 자동으로 1씩 증가시킨다.

---

## 4. 조회 (SELECT)

| 명령 | 역할 |
|------|------|
| `select * from emp;` | 전체 컬럼 조회 |
| `desc emp;` | 테이블 구조(describe) 확인 |
| `select * from emp where 조건;` | 조건 조회 |

![Desktop View](/assets/img/Database/SQL/Create-Table/6.png)
![Desktop View](/assets/img/Database/SQL/Create-Table/7.png)

**조건 조회 예:**

```sql
-- 부서번호가 20인 직원
select * from emp where deptno = 20;

-- 커미션이 NULL이고 급여 4000 이하
select * from emp where comm is NULL and sal <= 4000;
```

![Desktop View](/assets/img/Database/SQL/Create-Table/8.png)
![Desktop View](/assets/img/Database/SQL/Create-Table/9.png)

> 💡 `where comm is NULL`처럼 **NULL 비교는 `is`**를 쓴다(`= NULL` 아님). SQL은 마치 영어 기초 회화처럼 읽힌다.

---

## 📝 정리

```
테이블 생성·삽입·조회
├─ DB     show/create/use databases
├─ 생성   create table (컬럼 타입, primary key)
├─ 삽입   insert into ~ values(...)
└─ 조회   select * / desc / where 조건 (NULL은 is)
```

| 개념 | 한 줄 정의 |
|------|------|
| **create table** | 컬럼과 타입으로 테이블 정의 |
| **primary key** | 중복 불가한 식별 컬럼 |
| **desc** | 테이블 구조 확인 |
| **where ~ is NULL** | NULL 조건 조회 |

DB의 기본 흐름은 **"DB 생성 → use로 선택 → 테이블 생성 → insert → select"**다. 특히 조회에서 `where`와 `is NULL`만 익히면, 원하는 데이터를 자유롭게 뽑아낼 수 있다.
