---
title: "[SQL] 연습문제(생성 삽입 조회)"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

저번엔 MySQL을 배웠고, 이번엔 **Oracle**을 사용해본다. 오라클의 **사용자·권한 체계**를 이해하고, 서브쿼리를 활용한 조회 연습문제를 풀어본다.

> 윈도우에서 `sql`을 검색하면 **SQL*Plus**가 나온다.

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/1.png)

---

## 1. 오라클 사용자·권한 체계

오라클은 계층적인 사용자 구조를 가진다.

```
sys (SYSDBA)      ← 최상위 (직접 작업 어려움)
 └─ system        ← 관리자(간부급)
      └─ batman   ← 실제 작업할 사용자
```

| 사용자 | 역할 |
|------|------|
| `sys as sysdba` | 최상위(최고 관리자) |
| `system` | 관리자, 사용자·권한 관리 |
| `batman` | 실제 작업 사용자 |

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/4.png)

### 테이블스페이스와 사용자 생성

MySQL의 database에 해당하는 것이 오라클에선 **테이블스페이스(tablespace)**다.

```sql
-- (system으로) bigdata 테이블스페이스 생성 (XE 폴더에 bigdata.dbf, 5MB)
-- batman 사용자 생성 (비번 1234, bigdata를 기본, unlimited 권한)
```

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/7.png)

### 권한 부여

`batman`으로 접속하면 **권한 부족** 에러가 난다. 관리자로 권한을 줘야 한다.

```sql
connect system/1234
grant create session, create table to batman;   -- 접속·테이블 생성 권한
```

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/8.png)

> 💡 MySQL에서 쓴 명령어가 오라클에서도 똑같이 동작하는 이유: **미국 표준협회(ANSI)**가 DB 명령어를 표준화해두었기 때문이다. 그래서 어떤 DBMS든 기본 명령어는 같다.

---

## 2. spool — 결과 저장

```sql
-- spool [경로/파일명]  ... 쿼리들 ...  spool off
```

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/10.png)

> 💡 `spool`과 `spool off` 사이에 실행한 코드·결과가 **메모장 형식으로 저장**된다.

---

## 3. 서브쿼리 연습문제

### ① BLAKE보다 급여가 적은 사람

```sql
select ename, sal from emp
where sal < (select sal from emp where ename = 'BLAKE');
```

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/11.png)

> 💡 소괄호 안 **서브쿼리**로 BLAKE의 급여를 조회한 뒤, 그것을 비교 기준으로 삼는다.

### ② SCOTT의 사수보다 급여를 많이 받는 사원

```sql
select ename, sal from emp
where sal > (
    select sal from emp
    where empno = (select mgr from emp where ename = 'SCOTT')
);
```

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/12.png)

> 💡 서브쿼리를 **중첩**했다. `mgr`은 그 사람의 사수 사원번호다. ① SCOTT의 mgr(사수 번호) 조회 → ② 그 번호의 급여 조회 → ③ 그보다 많이 받는 사원 조회.

---

## 📝 정리

```
오라클 연습
├─ 사용자    sys > system > batman (계층)
├─ 권한      grant 권한 to 사용자
├─ 저장소    tablespace (MySQL의 database)
├─ spool     쿼리·결과를 파일로 저장
└─ 서브쿼리   (select ...)를 조건으로 중첩 활용
```

| 개념 | 한 줄 정의 |
|------|------|
| **sys/system/batman** | 오라클 사용자 계층 |
| **grant** | 권한 부여 |
| **tablespace** | 오라클의 데이터 저장 공간 |
| **서브쿼리** | 쿼리 안의 쿼리 (조건에 활용) |

오라클은 MySQL과 명령어(ANSI 표준)는 같지만 **사용자·권한 체계**가 특징이다. 그리고 **서브쿼리**를 중첩하면 "다른 조회 결과를 기준으로" 원하는 데이터를 정교하게 뽑아낼 수 있다.
