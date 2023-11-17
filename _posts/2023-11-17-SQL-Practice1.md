---
title: 연습문제(생성 삽입 조회)
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

저번 시간에는 Mysql을 배웠습니다. 이번 시간에는 Oracle database를 사용해보겠습니다~

오라클을 설치하고 윈도우 창에 sql이라고 치면 이렇게 sqlplus가 나옵니다!

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/1.png)

실행하겠습니다

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/2.png)

사진 설명을 입력하세요.

제가 사용했던 cmd입니다. 하나씩 뜯어보겠습니다.

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/3.png)

이것도 mysql처럼 처음에 패스워드에 더해 유저네임까지 입력해야합니다. 처음 우리는 아무것도 없으니까 관리자로 들어가야합니다. 그게 sys(system)입니다. 그럼 패스워드가 뜨는데 아무거나 눌러도 에러가 나옵니다. 에러 내용을 참고하여

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/4.png)

이렇게 들어갔습니다. sys as sysdba는 이 데이터베이스 세계에서 최상위 포식자라고 생각하시면 됩니다. 이 유저로는 직접 일을 진행하기 어렵습니다. 따라서 그 하위 단계인 system을 또 만들어야합니다.

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/5.png)

비번은 초기 설정때 만든 1234로 합니다. 잘 접속이 되었네요. 이제 데이터데이스를 만들어봅시다. 이름은 역시 bigdata로 하겠습니다~

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/6.png)

이렇게 입력하면 됩니다. 여긴 database가 아닌 tablespace로 합니다 ㅎ 그리고 파일 위치를 오라클이 설치된 파일에서 XE폴더를 찾아 bigdata.dbf로 합니다. 많이 쓰지는 않을거라 용량은 5mb로 했습니다~

이제 bigdata라는 행성이 만들어졌습니다. 이제 이 행성을 사용할 사용자를 또 만들어야합니다. 언제까지 system이라는 간부급이 움직일 수는 없으니 말이죠. DB에서의 국룰 사용자이름 'batman'으로 하겠습니다 ㅎㅎ

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/7.png)

이름을 정하고 비번도 1234입니다. bigdata를 디폴트로 하고 unlimited, 제한없는 권한을 주었습니다. 유저가 잘 생성되었군요. 이제 이 배트맨으로 bigdata라는 DB에서 여러가지 일을 할겁니다.

그럼 이제 이 배트맨으로 접속을 해봅시다.

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/8.png)

? 권한이 부족하다네요. 생성할 수 있는 권한을 system으로 주어야 할거 같습니다. 지금은 배트맨으로 접속했으니 다시 관리자로 접속해서 배트맨에게 권한을 줘봅시다.

```
connect system/1234
grant create session, create table to batman;
```

이렇게 세션과 테이블을 생성할 수있는 권한을 배트맨에게 줍니다. 그리고 다시 배트맨으로 접속합시다 ^^

이제 테이블을 만들건데 저번에 오라클에서 배포한 자료를 다시 쓰면 됩니다 ㅎㅎ 어떻게 mysql에서 쓴거를 똑같이 오라클에 써도 적용이 될까요?

미국 표준협회(ANSI)에서 데이터베이스 명령어를 다 정해놓았거든요. 그래서 어떤 프로그램을 사용하던지 명령어는 다 같습니다~

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/9.png)

내용을 담고 전부 select하였습니다. mysql이랑은 조금 레이아웃이 다르군요 ㅎㅎ 재밌습니다

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/10.png)

이렇게 spool을 적고 위치를 지정한 다음 파일이름까지 지정합니다.

그리고 나중에 spool off를 하면 그 사이에 있던 코드들이 모두 메모장 형식으로 저장이 됩니다~

몇가지 간단한 문제를 풀어보죠

// BLAKE 보다 급여가 적은 사람들의 이름과 급여를 적었습니다.

소괄호()안에는 다른 조건을 넣어서 블레이크의 샐러리를 조회 한다음 그것을 다시 비교척도로 사용한 뒤 이름과 급여를 조회하도록 하였습니다.

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/11.png)

//SCOTT의 사수보다 급여를 많이 받는 사원들의 이름과 급여를 조회합시다

![Desktop View](/assets/img/Database/SQL/Practice1-Insert-Create-Select/12.png)

좀 소괄호가 중첩이 많죠?mgr에 써있는 번호가 그 사람의 사수의 사원번호입니다. SCOTT의 mgr를 조회하고 또 그 번호의 사람의 급여를 조회한뒤 비교하였습니다~

재밌습니다 !

이번시간에는 여기까지 하겠습니다!