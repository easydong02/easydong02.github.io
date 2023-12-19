---
title: "[SQL] 여러가지 함수"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

오랜만의 sql 시간이네요 ㅎㅎ 반갑습니다! 이번 시간엔 여러가지 함수를 다뤄보겠습니다! 작업은 sqldeveloper 에서 진행했습니다!

```
# 숫자 관련 함수

ROUND 지정한 자리에서 반올림하는 함수.

1  2  3  .  4  5  6  7
-3 -2 -1    0  1  2  3


SELECT SALARY 월급,ROUND(SALARY/30,0) 일급 
FROM employees
```

round()는 소수 첫째 자리를 0으로 했을 때 지정한 자리에서 반올림하는 함수입니다

![Desktop View](/assets/img/Database/SQL/Functions/1.png)

0으로 하니까 0에서 반올림하여 일의자리 숫자부터 나오네요!

```
# 날짜
날짜에 더하거나 빼면 날짜가 계산된다.

SELECT TO_CHAR(SYSDATE,'YY/MM/DD'),
        SYSDATE+1 TOMORROW,
        SYSDATE-1 YESTERDAY
FROM DUAL;
```

TO\_CHAR는 인자의 값을 문자열로 바꿔줍니다. 따라서 TOMORROW와 YESTERDAY는 DATE지만 이 함수 안에서 나오는 값은 아니겠죠?

![Desktop View](/assets/img/Database/SQL/Functions/2.png)

잘 나오네요 ㅎㅎ

```
SELECT HIRE_DATE,
        ADD_MONTHS(HIRE_DATE,3), #월 날짜 수정
        ADD_MONTHS(HIRE_DATE,-3)
FROM employees
WHERE EMPLOYEE_ID BETWEEN 100 AND 110;
```

ADD\_MONTHS는 날짜의 값에서 월을 조정합니다.

![Desktop View](/assets/img/Database/SQL/Functions/3.png)

기존의 HIRE\_DATE에서 수정이 잘 되었네요 ㅎㅎ 그리고 WHERE문으로 조건까지 주어서 출력했습니다.

```
# 요일계산
일요일이 1, 월요일은 2...
SELECT HIRE_DATE,
        NEXT_DAY(HIRE_DATE,1), #가까운 일요일
        NEXT_DAY(HIRE_DATE,6)  #가까운 금요일
FROM EMPLOYEES;
```

NEXT\_DAY()는 넣은 날짜로 부터 가까운 요일을 계산해줍니다. 일요일의 1부터 순서대로입니다.

![Desktop View](/assets/img/Database/SQL/Functions/4.png)

잘 나오네요 ㅎㅎ

```
# LAST_DAY 이번 달 마지막 날짜

SELECT HIRE_DATE,
    LAST_DAY(HIRE_DATE) 결과
FROM EMPLOYEES;
```

LAST\_DAY()는 입력한 날짜의 월 값에 가장 마지막 일수를 보여줍니다.

![Desktop View](/assets/img/Database/SQL/Functions/5.png)

잘 나오네요 ㅎㅎ

```
# 날짜 반올림

SELECT HIRE_DATE,
    ROUND(HIRE_DATE,'MONTH') 결과

FROM EMPLOYEES;
```

ROUND()에서 'MONTH'를 준다면 그 달 중 일수를 반으로 나누어 반올림을 계산합니다

![Desktop View](/assets/img/Database/SQL/Functions/6.png)

잘 나오네요 ㅎㅎ 재밌습니다!

\# 변환 함수...

각 열에 대해 데이터 타입이 규정된다. 그러므로 SQL문을 실행하기 위해 데이터 값의 데이터 타입을 변환할 때도 있습니다.

이 때 사용하는 함수를 변환함수라고 합니다.

데이터 타입의 변환은 자동으로(암묵적으로) 또는 사용자에 의해 수동으로(명시적으로)변환됩니다.

```
SELECT 1+'2'
FROM DUAL;
```

![Desktop View](/assets/img/Database/SQL/Functions/7.png)

2는 분명히 문자열인데 자동으로 변환이 되어서 숫자 계산이 되었습니다. 그래도 안정성을 위해 수동변환하는게 조금더 좋겠죠?

```
SELECT TO_CHAR(SYSDATE,'YY'),
        TO_CHAR(SYSDATE,'YYYY'),
        TO_CHAR(SYSDATE,'MM'),
        TO_CHAR(SYSDATE,'YYMMDD')
FROM DUAL;
```

TO\_CHAR()는 문자열로 바꿔주죠? 그래서 SYSDATE로 이 컴퓨터 시간을 가져와서 옆의 형식대로 출력하게 했습니다

![Desktop View](/assets/img/Database/SQL/Functions/8.png)

잘 나오네요 ㅎㅎ

```
SELECT TO_CHAR(SYSDATE,'HH:MI:SS PM'),
        TO_CHAR(SYSDATE,'YYYY/MM/DD HH:MI:SS PM')
FROM DUAL;
```

![Desktop View](/assets/img/Database/SQL/Functions/9.png)

이런식으로도 표현할 수 있습니다

```
SELECT TO_DATE('20211005','YYMMDD')
FROM DUAL;
```

TO\_DATE는 반대로 날짜형식으로 바꿔줍니다

![Desktop View](/assets/img/Database/SQL/Functions/10.png)

잘 나오네요 ㅎㅎ

```
SELECT FIRST_NAME,
        DEPARTMENT_ID,
        SALARY 원래급여,
        DECODE(DEPARTMENT_ID, 60,SALARY* 1.1,SALARY),
        DECODE(DEPARTMENT_ID,60,'10%인상','현상 유지')
FROM EMPLOYEES;
```

DECODE는 SQL에서 IF와 같은 것이라고 보면 됩니다.

DECODE(열,조건,치환(TRUE),기본값(FALSE)) 입니다

따라서 해석하자면 DEPARTMENT\_ID가 60이면 SALARY에서 1.1을 곱하고 아니면 그대로 값을 변경합니다.

![Desktop View](/assets/img/Database/SQL/Functions/11.png)

잘 나오네요 ㅎㅎ

```
SELECT SALARY * NVL(COMMISSION_PCT,1) # NULL이면 SALARY에 1곱함
FROM EMPLOYEES
ORDER BY COMMISSION_PCT;
```

NVL()은 NULL값을 처리할 때 유용합니다. 해석하자면 SALARY에 COMMISSION\_PCT를 곱하는데 NVL()안에 있죠? 따라서 이게 널이면 1로 변환합니다.

![Desktop View](/assets/img/Database/SQL/Functions/12.png)

이제 널값이 안나오죠?ㅎㅎ 재밌네요

이번시간에는 여기까지 하겠습니다!!