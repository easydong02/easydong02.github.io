---
title: "[SQL] 시퀀스 조인 외래키2"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---
저번 시간에는 emp, dept테이블을 그냥 조건문 where로 연결시켰죠? 이번엔 아예 테이블 2개를 만드는데 foreign key로 두 테이블을 시스템적으로 연결시켜보겠습니다.

저는 subject 테이블과 category 테이블을 만들어서 두개를 이어줄 foreign key를 지정하겠습니다. 그리고 각각 primary key가 있어야 하니까 각각 시퀀스도 만들어줍니다.

먼저 우리의 batman으로 접속하죠

```
create table category(

 category_id number primary key

 , category_name varchar(25)

 );
```

category 테이블입니다. category\_id 를 프라이머리 키로 설정하고 카테고리 이름을 만들었습니다. 그리고 이 카테고리 id를 foreign키로 subject 테이블을 만들 때 사용하려고 합니다.

이제 subject 테이블을 만듭시다.

```
create table subject(

 subject_id number primary key

 , category_id number

 , subject_name varchar(15)

, target varchar(8)

, constraint fk_category_subject foreign key(category_id) references category(category_id)

 );
```

서브젝트 테이블엔 먼저 subject\_id라는 프라이머리 키를 설정하고 이제 category\_id 를 설정합니다. 그리고 서브젝트 이름과 타겟이라는 필드를 각각 만들었습니다. 그리고 마지막에 제약조건 처럼

```
constraint fk_category_subject foreign key(category_id) references category(category_id)
```

가 있는데요 fk는 foreign key입니다. 그리고 fk\_category\_subject는 foreign key로 두 테이블을 연결하겠다는 것이죠. 그리고 foreign key는 category\_id이고 이것은 부모테이블이라 부를 수 있는 category의 category\_id를 참조한다는 뜻입니다.!

이번 시간은 엄청 짧았네요 ㅎㅎ 여기까지 하도록 하겠습니다!