---
title: "[SQL] 시퀀스 조인 외래키"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

이번엔 자동증가 sequence와 정규화 작업에 꼭 필요한 join과 그 축을 담당하는 foreign key에 대해서 알아보겠습니다!

**시퀀스(Sequence)**

mysql에서는 자동증가옵션이 있었죠? 그런데 오라클엔 없습니다. 그래서 따로 자동적으로 증가하는 sequence를 만들어주어야 하죠.

저번에 만들었던 사용자 batman을 접속하고 시퀀스를 바로 만들 수 있을까요? 아닙니다. 이것도 마찬가지로 관리자에 의해 시퀀스를 만들 수 있는 권한을 받아야 하죠. 문법은 똑같습니다. 관리자인 system으로 접속한 뒤,

```
grant create sequence to batman;
```

이제 다시 배트맨으로 접속하겠습니다. 이제 batman도 시퀀스를 만들 수 있죠.

```
create sequence seq
increment by 1
start with 1;
```

이겁니다. seq라는 이름의 시퀀스를 만들고 increment by1 은 1씩증가입니다. 그리고 start with1은 1부터 시작이죠. 이제 seq는 1부터 시작하여 1씩 증가하는 시퀀스입니다. 이거를 어디다 쓰냐고요? 테이블을 만들 때 넣어주면 됩니다.

테이블을 하나 만듭시다.

```
 1  create  table  family(
  2    family_id  number primary key 
  3    , name varchar(25)
  4    , age number 
  5    , job  varchar(25)
  6* )
```

family 라는이름의 테이블이고 family\_id는 number(오라클에선 int 대신 number를 씁니다)타입이고 primary key입니다. 그리고 이름과 나이 job을 적어주었죠. 이제 이 테이블에 하나씩 레코드를 추가할 건데 어떻게 하는지 봅시다.

```
insert into family(family_id, name, age, job)
 values(seq.nextval,'spiderman',78,'police');
```

insert into family는 다 아실테고, 여기서 중요한건 family\_id에 어떤 값을 넣었나 보죠. number타입이라 원래는 일일이 값을 넣어줘야하지만 이제는 아까만든 seq라는 자동증가 시퀀스가 있습니다. 근데 옆에 .nextval이 있네요? 이건 마지막 순번뒤에 차례대로 하나씩 넣는것입니다. 이 테이블엔 아직 아무 레코드도 없으니 마지막 순번이 0일테니까 1을 넣겠죠? 이런식으로 사용합니다. 사용방법은 다소 복잡하지만 잘 알면 1씩 증가가 아니라 자신만의 방식대로 자동증가 시퀀스를 만들 수 있습니다.

**join**

join은 두 테이블 을 이어주는 작업입니다. 그리고 foreign key를 축으로 서로 이어주는 것이죠. 이제 오라클 emp와 dept테이블을 봅시다.

![Desktop View](/assets/img/Database/SQL/Seq-Join-ForeignKey/1.png)

여기서 우리는 deptno를 중심으로 두 테이블을 연결하고 싶습니다. 일단 두 테이블을 같이 불러볼까요?

```
select * from emp,dept;
```

![Desktop View](/assets/img/Database/SQL/Seq-Join-ForeignKey/2.png)

헐 엄청 많은 것들이 나오네요. 맨 밑에 보시면 56 rows selected.라는게 보이시죠?

알고보면 emp테이블의 14개의 레코드와 dept의 4개의 레코드가 서로 서로 중복연결되어 14\*4가 된 것이죠.

그럼 이것을 deptno가 중복되지 않게 하나씩 연결하려면 어떻게 하냐면

```
select * from emp,dept where emp.deptno=dept.deptno;
```

입니다 여기서 두 테이블을 이어주는 외부키는 deptno겠죠? 이제 두 테이블의 모든 레코드들을 선택할 때 조건을 겁니다. 두 테이블의 각 레코드의 deptno가 같은 것만 출력하는 것이죠 해볼까요?

![Desktop View](/assets/img/Database/SQL/Seq-Join-ForeignKey/3.png)

이제 14개 의 레코드로 보이네요 보시면 중간에 deptno를 중심으로 두 테이블이 잘 연결이 된 것을 볼 수 있습니다.

이번 시간엔 여기까지 하겠습니다~