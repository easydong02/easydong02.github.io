---
title: "[SQL] DML DCL DDL 집계함수"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 SQL의 세 가지 언어 분류 **DML·DDL·DCL**과 **집계함수**를 정리한다. 이 명령어들은 **ANSI 표준 쿼리**라 대부분의 DBMS에서 공통으로 쓸 수 있다.

> SQL은 목적에 따라 세 부류로 나뉜다.

| 분류 | 이름 | 명령어 |
|------|------|------|
| **DML** | 데이터 조작어 | `INSERT` `UPDATE` `DELETE` `SELECT` |
| **DDL** | 데이터 정의어 | `CREATE` `ALTER` `DROP` |
| **DCL** | 데이터 제어어 | `GRANT` `REVOKE` (권한) |

---

## 1. DML — 데이터 조작

### INSERT — 레코드 추가

```sql
insert into emp(empno, ename, sal) values(7777, 'Dong Hee', 40000);
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/2.png)
![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/3.png)

> 💡 지정한 컬럼(번호·이름·급여)만 넣으면 **나머지는 NULL**로 채워진다.

### DELETE — 레코드 삭제

```sql
delete from emp where ename = 'MILLER';
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/4.png)

> ⚠️ `WHERE`를 빼면 **전체 레코드가 삭제**되니 주의.

### UPDATE — 레코드 수정

```sql
UPDATE EMP SET JOB = 'PRESIDENT', SAL = 5000 WHERE EMPNO = 7369;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/5.png)

`SET`으로 여러 컬럼을 한 번에 바꾼다. (SMITH가 벼락 진급!)

---

## 2. DDL — 데이터 정의

`CREATE`(생성) 외에 **테이블 구조 자체**를 바꾸는 `ALTER`가 강력하다.

### ALTER — 컬럼 삭제/추가

```sql
ALTER TABLE EMP DROP COLUMN COMM;      -- 컬럼 삭제
ALTER TABLE EMP ADD phone char(13);    -- 컬럼 추가
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/6.png)
![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/7.png)

| 명령 | 역할 |
|------|------|
| `ALTER TABLE ~ DROP COLUMN` | 컬럼(열) 자체 삭제 |
| `ALTER TABLE ~ ADD` | 컬럼 추가 (값은 NULL로 시작) |

> ⚠️ DDL은 **테이블 구조를 바꾸는** 강력한 명령어라 신중히 써야 한다.

---

## 3. DCL — 권한 제어

> DCL은 **권한을 주고(GRANT) 뺏는(REVOKE)** 것이다. 오라클에서 특정 사용자(BATMAN)에게 권한을 부여했던 것이 이에 해당한다.

---

## 4. 집계함수

여러 행을 하나의 값으로 요약한다.

| 함수 | 역할 |
|------|------|
| `COUNT()` | 총 개수 |
| `AVG()` | 평균 |
| `MAX()` / `MIN()` | 최대 / 최소 |
| `SUM()` | 합계 |

```sql
SELECT COUNT(EMPNO) AS 총사원수 FROM EMP;
SELECT AVG(SAL) AS 평균급여 FROM EMP;
SELECT MAX(SAL) AS 최대값 FROM EMP;
SELECT MIN(SAL) AS 최소값 FROM EMP;
SELECT SUM(SAL) AS 급여합 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/8.png)

> 💡 **`AS`**는 결과 컬럼에 **별칭**을 붙인다. 별칭이 없으면 `COUNT(EMPNO)`처럼 그대로 나오므로, 가독성을 위해 사용한다.

---

## 5. 정렬·그룹화

```sql
SELECT * FROM EMP ORDER BY SAL DESC;   -- 내림차순
SELECT * FROM EMP ORDER BY SAL ASC;    -- 오름차순
SELECT DEPTNO FROM EMP GROUP BY DEPTNO; -- 그룹화(중복 제거)
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/13.png)
![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/15.png)

| 절 | 역할 |
|------|------|
| `ORDER BY 컬럼 DESC/ASC` | 내림/오름차순 정렬 |
| `GROUP BY 컬럼` | 중복 값을 묶어 종류별로 |

---

## 📝 정리

```
SQL 3분류 + 집계
├─ DML  INSERT/UPDATE/DELETE/SELECT (데이터 조작)
├─ DDL  CREATE/ALTER/DROP (구조 정의)
├─ DCL  GRANT/REVOKE (권한 제어)
├─ 집계  COUNT/AVG/MAX/MIN/SUM (+ AS 별칭)
└─ 정렬  ORDER BY(정렬) / GROUP BY(그룹화)
```

| 개념 | 한 줄 정의 |
|------|------|
| **DML/DDL/DCL** | 조작 / 구조 정의 / 권한 |
| **AS** | 결과 컬럼 별칭 |
| **ORDER BY / GROUP BY** | 정렬 / 그룹화 |

SQL은 목적에 따라 **DML(조작)·DDL(정의)·DCL(제어)**로 나뉜다. 특히 DDL은 구조를 바꾸는 강력한 명령어이니 신중히, `WHERE` 없는 DELETE/UPDATE는 항상 조심하자.
