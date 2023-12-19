---
title: "[SQL] 테이블 생성 삽입 조회"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---
데이터베이스 시간입니다!

데이터베이스란 우리가 흔히 알고 있는 엑셀과 같은 어떠한 정보들의 유의미한 집합을 종류로 나누어 저장하는 창고같은 것입니다.

DB에는 많은 것들이 있지만 먼저 Mysql을 배웠습니다. 흥미롭더군요 ㅎㅎ 시작하겠습니다.

먼저 오라클홈페이지에서 Mysql을 다운받습니다.

![Desktop View](/assets/img/Database/SQL/Create-Table/1.png)

그럼 탐색기에 my만 쳐도 저런 Mysql cmd가 있네요. 설치할 때 생성한 비밀번호를 누르면

![Desktop View](/assets/img/Database/SQL/Create-Table/2.png)

이런식의 cmd창이 나옵니다. 명령어를 하나씩 배워보도록 하죠. 먼저

**show databases;** 입니다. 이것은 어떤 DB가 있는지 알려주죠.

![Desktop View](/assets/img/Database/SQL/Create-Table/3.png)

저렇게 깔끔하게 DB목록들이 나오는 것을 확인할 수 있습니다. 이제 우리만의 DB를 만들어봅시다.

**create database myDB;** 명령어는 myDB라는 이름의 DB를 생성하는 겁니다.

![Desktop View](/assets/img/Database/SQL/Create-Table/4.png)

입력을 하면 만들어졌습니다. 그런데 안에는 아무것도 없겠죠? 이제 테이블을 한 개 만들어 봅시다. 만들기 전에 우리가 만든 DB를 선택을 먼저 해야 그 안에서 테이블을 만들겠죠? 마치 이클립스에서 패키지와 클래스같은 개념이라고 보시면 됩니다.

**use myDB;** 이 명령어로 myDB라는 DB를 사용합니다. 이제 테이블을 만들건데 저는 히어로의 이름과 나이와 능력을 포함하는 테이블을 만들고싶습니다.

```
create table heros(

name char(20)

, age int

, ability char(20)

) default character set utf8;

```

좀 길죠? 먼저 heros라는 이름의 테이블이고 그 안에는 name이라는 문자(char)타입의 종류칸을 만들었습니다 (20)은 뭐냐구요? 용량입니다. char문자 하나당 2바이트이니까 10글자까지 잡을 수 있죠. 그리고 콤마(,)로 int 타입의 age, 또 char타입의 ability를 만들고 디폴트 charset은 utf8로 했습니다. 왜냐하면 우리는 한글을 사용할거기 때문이죠. 이제 그 테이블에 레코드를 하나씩 넣어보겠습니다!

```
insert into heros(name, age, ability) values('배트맨',32,'날기');
```

로 해주면 됩니다. 이제 추가가 되었는데 확인을 해볼까요?

```
select name, age, job from heros;
```

라는 명령어로 우리가 넣었던 name, age, ability를 확인해봅시다.

![Desktop View](/assets/img/Database/SQL/Create-Table/5.png)

이렇게 또 깔끔하게 나오는군요! 재밌습니다. 이제 오라클에서 학습하라고 배포한 연습용 DB를 봅시다

```
create table dept(

deptno int primary key auto_increment

,dname varchar(14)

,loc varchar(13)

 );

insert into dept(deptno,dname,loc) values(10,'ACCOUNTING','NEW YORK');

insert into dept(deptno,dname,loc) values(20,'RESEARCH','DALLAS');

insert into dept(deptno,dname,loc) values(30,'SALES','CHICAGO');

insert into dept(deptno,dname,loc) values(40,'OPERATIONS','BOSTON');


create table emp(

empno int primary key auto_increment

,ename varchar(10)

 ,job varchar(9)

,mgr int

,hiredate timestamp

,sal int

,comm int

,deptno int

);


insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7369,'SMITH','CLERK',7902,'80/12/17',800,20);

insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(7499,'ALLEN','SALESMAN',7698,'81/02/20',1600,300,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(7521,'WARD','SALESMAN',7698,'81/02/22',1250,500,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7566,'JONES','MANAGER',7839,'81/04/02',2975,20);

insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(7654,'MARTIN','SALESMAN',7698,'81/09/28',1250,1400,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7698,'BLAKE','MANAGER',7839,'81/05/01',2850,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7782,'CLARK','MANAGER',7839,'81/06/09',2450,10);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7788,'SCOTT','ANALYST',7566,'87/04/19',3000,20);

insert into emp(empno,ename,job,hiredate,sal,deptno) values(7839,'KING','PRESIDENT','81/11/17',5000,10);

insert into emp(empno,ename,job,mgr,hiredate,sal,comm,deptno) values(7844,'TURNER','SALESMAN',7698,'81/09/08',1500,0,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7876,'ADAMS','CLERK',7788,'87/05/23',1100,20);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7900,'JAMES','CLERK',7698,'81/12/03',950,30);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7902,'FORD','ANALYST',7566,'81/12/03',3000,20);

insert into emp(empno,ename,job,mgr,hiredate,sal,deptno) values(7934,'MILLER','CLERK',7782,'82/01/23',1300,10);
```

뭐가 엄청 많죠? 근데 자세히 보시면 테이블 2개를 만들고 거기에 레코드를 삽입한 것입니다.

새로운 명령어를 알려드릴게요

**select \*** 입니다. 전체 필드를 선택하여 볼 수 있죠. 보니까 테이블 이름이 dept(부서)와 emp(직원)인거 보니까 감이 딱 오죠?

![Desktop View](/assets/img/Database/SQL/Create-Table/6.png)

잘 나오네요 ㅎㅎ 깔끔합니다. 이제 저 테이블의 속성을 볼 수있는 명령어가 있습니다.

desc(describe입니다) emp;

![Desktop View](/assets/img/Database/SQL/Create-Table/7.png)

필드와 그 필드의 타입과 여러가지를 알 수 있습니다!

이제 조건을 통해서 데이터를 추출해봅시다.

직원들 중에 부서번호가 20인 사람들을 추출합니다.

```
select * from emp where deptno=20;
```

![Desktop View](/assets/img/Database/SQL/Create-Table/8.png)

잘 나오네요 ~ 재밌습니다 ㅎㅎ

이제 직원들 중에 커미션이 NULL 이고 샐러리가 4000이하인 사람들을 추출합시다

```
select * from emp where comm is NULL and sal<=4000;
```

이렇게 쓰면 됩니다. 영어 기초회화처럼 쉽죠?

![Desktop View](/assets/img/Database/SQL/Create-Table/9.png)

잘 추출되는 것을 확인할 수 있습니다!! 재밌네요 ㅎㅎ

오늘은 여기까지 하겠습니다!