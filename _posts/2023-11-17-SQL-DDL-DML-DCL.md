---
title: DML DCL DDL 집계함수
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

이번 시간에는 Mysql의 DML, DDL, DCL에 대해서 알아보겠습니다! 이 언어들은 ANSI쿼리익 때문에 모든 DBMS에서 사용이 가능합니다.

**DML(Database Manipulation Language)**

DML은 데이터 조작어입니다. 여기에는 CREATE, ALTER, DROP이 있습니다. 하나씩 보겠습니다.

**INSERT**

INSERT는 존재하는 테이블에 레코드를 추가하는 것입니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/1.png)

저번에 오라클 덤프 테이블을 전체 조회했습니다. 이제 여기에 삽입을 하나 해보겠습니다.

```
insert into emp(empno, ename,sal) values(7777,'Dong Hee',40000);
```

저는 EMP테이블에 번호와 이름과 급여만 넣고싶습니다. 그리고 VALUES에 직접 입력했습니다. 번호는 7777, 이름은 'DONG HEE', 급여는 40000로 크게 잡았습니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/2.png)

잘 되었네요 ㅎㅎ 다시 테이블을 봐볼까요?

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/3.png)

중간에 DONG HEE가 잘 있습니다. 입력 안 한 나머지는 NULL이라고 나오네요 ㅎㅎ 재밌습니다.

**DELETE**

DELETE는 존재하는 테이블에 요소를 삭제하는 명령어입니다. 바로 써볼게요. 저는 예전부터 MILLER가 맘에 안들었습니다.. 그냥 ㅋㅋ 그래서 퇴사시켜보겠습니다.

```
delete from emp where ename='MILLER';
```

이름이 밀러인 사람을 EMP에서 지우는 명령어 입니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/4.png)

그는 갔습니다..ㅜㅜ 뭐 아무튼 작동이 잘 되는 것을 볼 수 있었습니다.

**UPDATE**

UPDATE는 존재하는 테이블에 레코드를 수정하는 것입니다. 여기서 SMITH의 직급과 급여를 수정해보겠습니다.

```
UPDATE EMP SET JOB='PRESIDENT', SAL=5000 WHERE EMPNO=7369;
```

사원번호가 7369로 한담 JOB과 SAL을 저렇게 SET으로 바꿀 수 있습니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/5.png)

SMITH가 벼락진급을 해버렸습니다..ㅎㅎ 재밌군요.

**DDL(DATABASE DEFINITION LANGUAGE)**

DDL은 데이터 정의어라고도 하며 아주 강력한 명령어들입니다. 여기엔 CREATE, ALTER, DROP의 명령어가 있습니다. CREATE는 테이블과 데이터베이스들을 생성하는 명령어 입니다.

**ALTER**

ALTER는 테이블의 틀을 변경하는 것입니다. DROP과 함께 사용하여 필드 하나를 지워보겠습니다.

```
ALTER TABLE EMP DROP COLUMN COMM;
```

이것은 EMP 테이블에서 COMM이라는 열 자체를 지워버리는 명령어입니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/6.png)

보시면 COMM이라는 필드자체가 사라져 버렸습니다.. 무섭네요.

그리고 ADD를 같이 쓸 수 도 있습니다.

```
 ALTER TABLE EMP ADD phone char(13);
```

이것은 DROP과 반대로 필드를 생성하는 명령어입니다. phone이라는 문자가 들어가는 필드를 생성하겠습니다.

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/7.png)

PHONE이라는 필드가 생겼습니다. 값을 넣은 적이 없으니 모두 NULL이 뜨는 것을 알 수 있습니다.

**DCL(DATABASE CONTROL LANGUAGE)**

DCL은 권한을 주고 뺏는 것입니다. 저번에 오라클에서 BATMAN에게 권한을 주었었죠? 그런 겁니다 ㅎㅎ

**집계함수**

SQL도 나름 언어이기에 연산과 함수를 지원합니다. 그 중 집계를 도출할 때 사용되는 함수를 알아봅시다.

**COUNT() : 총개수**

COUNT()는 괄호 안에 있는 필드의 총 개수를 알려주는 함수입니다.

```
SELECT COUNT(EMPNO) AS 총사원수 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/8.png)

잘 나오네요 ^^ 여기서 AS는 그 뒤에 문자를 별칭으로 씁니다. 원래는 COUNT(EMPNO)라고 나오겠죠? 가독성을 위한겁니다.

**AVG() : 평균**

AVG()는 괄호 안에 있는 필드의 평균을 알려주는 함수입니다.

```
SELECT AVG(SAL) AS 평균급여 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/9.png)

**MAX() : 최대값**

MAX()는 괄호 안에 있는 필드의 최대값을 알려주는 함수입니다.

```
SELECT MAX(SAL) AS 최대값 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/10.png)

**MIN() : 최소값**

MIN()는 괄호 안에 있는 필드의 최소값을 알려주는 함수입니다.

```
SELECT MIN(SAL) AS 최소값 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/11.png)

**SUM() : 합계**

SUM()는 괄호 안에 있는 필드의 합계를 알려주는 함수입니다.

```
SELECT SUM(SAL) AS 급여합 FROM EMP;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/12.png)

**ORDER BY \* DESC : 내림차순**

ORDER BY \* DESC는 \* 안에 있는 필드 내림차순으로 알려주는 함수입니다.

```
SELECT * FROM EMP ORDER BY SAL DESC;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/13.png)

SAL을 기준으로 내림차순으로 해줍니다.

**ORDER BY \* ASC : 오름차순**

ORDER BY \* ASC는 \* 안에 있는 필드 오름차순으로 알려주는 함수입니다.

```
SELECT * FROM EMP ORDER BY SAL ASC;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/14.png)

**GROUP BY \* : 그룹화**

GROUP BY \* 는 \* 안에 있는 필드를 그룹화하여 알려주는 함수입니다. 중복된 값이 있으면 하나로 쳐서 어떤 종류가 있는지 쉽게 알 수 있습니다

```
SELECT DEPTNO FROM EMP GROUP BY DEPTNO;
```

![Desktop View](/assets/img/Database/SQL/DML-DDL-DCL/15.png)

이번 시간에는 여기까지 하겠습니다 ~