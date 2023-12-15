---
title: [SQL] 아우터 조인 트랜잭션
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

이번에는 아웃터 조인을 사용해볼게요 ㅎㅎ.

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/1.png)

이것을 보면 약간 의아한게 있습니다. 분명 dept테이블에는 40번의 운영 부서가 보스턴에 자리하고 있다는 정보는 있는데 막상 emp테이블에는 저 40번 부서에서 근무하는 사람이 없다 이거죠.. 그렇다고 join을 하면 그 기준이 deptno가 같은 것만 짝짓다 보니 40번부서는 아예 나오지도 않습니다. 그런데 우리가 원하는 것은 40번부서도 일단 표현을 하고 0명이 근무하던지.. 이런식으로 나타내는게 좋겠죠? 그래서 outer join을 써보겠습니다.

```
  1  select * 
  2  from  dept d  left outer join emp e
  3* on d.deptno = e.deptno 
```

평소에 쓰던 inner join과 다른 점은 일단 표현방식에서 left outer join이 보이네요. 이것은 dept d 를 왼쪽에 기준시키고 거기에 맞춰 emp 테이블을 붙이는 것입니다. 그리고 where도 아니고 on이 쓰였네요. 한번 해볼까요?

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/2.png)

오 역시 테이블이 잘 나오는 군요. 그런데 원래는 14줄이 나왔는데 이번엔 15줄이네요? 맞습니다. 바로 40번 Operations 부서가 나온 것입니다. 물론 여기에 매칭되는 emp테이블의 레코드는 하나도 없군요.. outer join은 이렇게 일단 기준이 되는 테이블의 모든 것을 보여주기는 합니다!

그런데 이렇게 자료를 쭉 펼쳐놓는 것보다 요약해서 보여줍시다!

```
  1  select  d.deptno, dname, count(e.deptno)
  2  from  dept d  left outer join emp e
  3  on d.deptno = e.deptno 
  4  group by d.deptno, dname
  5* order by  d.deptno asc
```

먼저 deptno와 dname을 그룹화해서 요약을 해봅시다. 그렇다면 select에는 올 수 있는 것이 제약이 되겠죠. 집계함수라던지 그룹화 한 것들을 선택합니다. 그래서 딱 그룹화 한것과 count(e.deptno)를 하여 그 부서에 몇명이 있는지 보여주고 싶습니다. 그리고 부서번호를 오름차순으로 정렬했습니다.

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/3.png)

잘 나옵니다!

**트랜스액션(TransAction)**

DML 작업시 세부업무가 모두 성공해야, 전체를 성공으로 간주하는 논리적 업무수행 단위입니다.

우리가 DML( delete,insert,update)할 때 그 수행이 완전히 저장되는 것이 아니라 rollback으로 다시 되돌릴 수 있습니다.

미연의 실수를 방지하는 것이죠. 그리고 그 체크포인트는 commit; 이라는 것입니다. 이거를 해야 확정이 되는 것이죠.

하지만 DDL같은 정의어는 그 작업을 수행할때 commit이 동반됩니다. 즉 수행 하나하나마다 그냥 확정이 되는 것이죠 그래서 쓸 때 조심해야 합니다!

이번 빅데이터시간에는 여기까지 하겠습니다~