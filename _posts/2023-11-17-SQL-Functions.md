---
title: "[SQL] 여러가지 함수"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

오랜만의 SQL 시간! 이번 글에서는 **숫자·날짜·변환·조건 함수**를 다룬다. (작업은 SQL Developer에서 진행)

---

## 1. 숫자 함수 — ROUND

```sql
SELECT SALARY 월급, ROUND(SALARY/30, 0) 일급
FROM employees;
```

![Desktop View](/assets/img/Database/SQL/Functions/1.png)

> 💡 `ROUND(값, 자리)`는 지정한 자리에서 반올림한다. `0`이면 소수 첫째 자리에서 반올림해 일의 자리까지 나온다.

```
자리:  ... -2 -1 [0] 1  2 ...
값:         1  2 . 3  4  5   (0은 소수점 기준)
```

---

## 2. 날짜 함수

날짜에 `+`, `-`를 하면 날짜가 계산된다.

| 함수 | 역할 |
|------|------|
| `SYSDATE` | 현재 시스템 날짜 |
| `SYSDATE+1` / `-1` | 내일 / 어제 |
| `ADD_MONTHS(날짜, n)` | n개월 더하기 |
| `NEXT_DAY(날짜, 요일)` | 가까운 요일 (일=1) |
| `LAST_DAY(날짜)` | 그 달의 마지막 날 |
| `ROUND(날짜, 'MONTH')` | 월 기준 반올림 |

```sql
SELECT TO_CHAR(SYSDATE,'YY/MM/DD'), SYSDATE+1 TOMORROW, SYSDATE-1 YESTERDAY
FROM DUAL;

SELECT HIRE_DATE, ADD_MONTHS(HIRE_DATE, 3), ADD_MONTHS(HIRE_DATE, -3)
FROM employees WHERE EMPLOYEE_ID BETWEEN 100 AND 110;

SELECT HIRE_DATE, NEXT_DAY(HIRE_DATE, 1), LAST_DAY(HIRE_DATE)
FROM EMPLOYEES;
```

![Desktop View](/assets/img/Database/SQL/Functions/3.png)
![Desktop View](/assets/img/Database/SQL/Functions/5.png)

> 💡 `DUAL`은 오라클의 **가상 테이블**로, 함수 결과만 확인할 때 쓴다.

---

## 3. 변환 함수

> 각 열에는 데이터 타입이 정해져 있어, SQL 실행 시 타입을 변환할 때가 있다. **자동(암묵적)** 또는 **수동(명시적)**으로 변환한다.

```sql
SELECT 1 + '2' FROM DUAL;   -- '2'가 자동으로 숫자로 변환 → 3
```

> ⚠️ 자동 변환도 되지만, **안정성을 위해 수동 변환**이 더 좋다.

| 함수 | 역할 |
|------|------|
| `TO_CHAR(값, 형식)` | 문자열로 변환 |
| `TO_DATE(값, 형식)` | 날짜로 변환 |

```sql
SELECT TO_CHAR(SYSDATE,'YYYY/MM/DD HH:MI:SS PM') FROM DUAL;
SELECT TO_DATE('20211005','YYMMDD') FROM DUAL;
```

![Desktop View](/assets/img/Database/SQL/Functions/9.png)

---

## 4. 조건 함수 — DECODE, NVL

### DECODE — SQL의 IF

```sql
SELECT FIRST_NAME, SALARY 원래급여,
       DECODE(DEPARTMENT_ID, 60, SALARY*1.1, SALARY),        -- 값
       DECODE(DEPARTMENT_ID, 60, '10%인상', '현상 유지')      -- 라벨
FROM EMPLOYEES;
```

> 💡 `DECODE(열, 조건, TRUE값, FALSE값)`. 부서가 60이면 급여 1.1배, 아니면 그대로. **정확히 일치**하는 조건에 쓴다.

![Desktop View](/assets/img/Database/SQL/Functions/11.png)

### NVL — NULL 처리

```sql
SELECT SALARY * NVL(COMMISSION_PCT, 1)   -- NULL이면 1로
FROM EMPLOYEES ORDER BY COMMISSION_PCT;
```

> 💡 `NVL(값, 대체값)`은 값이 **NULL이면 대체값**으로 바꾼다. 위는 커미션이 NULL이면 1을 곱해 급여가 그대로 유지되게 한다.

![Desktop View](/assets/img/Database/SQL/Functions/12.png)

---

## 📝 정리

```
SQL 함수
├─ 숫자   ROUND(값, 자리)
├─ 날짜   SYSDATE, ADD_MONTHS, NEXT_DAY, LAST_DAY
├─ 변환   TO_CHAR(문자), TO_DATE(날짜)
└─ 조건   DECODE(IF), NVL(NULL 처리)
```

| 함수 | 한 줄 정의 |
|------|------|
| **ROUND** | 지정 자리 반올림 |
| **TO_CHAR / TO_DATE** | 문자 / 날짜 변환 |
| **DECODE** | SQL의 IF (정확 일치) |
| **NVL** | NULL을 대체값으로 |

SQL도 하나의 언어라 다양한 내장 함수를 제공한다. 특히 **DECODE(조건 분기)**와 **NVL(NULL 처리)**는 실무 쿼리에서 끝없이 쓰이니 확실히 익혀두자.
