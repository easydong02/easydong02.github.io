---
title: "[SQL] 아우터 조인 트랜잭션"
date: 2023-11-17 00:00:00 +0900
categories: [Database, SQL]
tags: [oracle, sql]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **아우터 조인(Outer Join)**과 **트랜잭션(Transaction)**을 정리한다.

> **문제 상황**: `dept`에는 40번(운영, 보스턴) 부서가 있지만, `emp`에는 40번 부서 직원이 없다. 일반 조인(inner join)은 **양쪽에 매칭되는 것만** 짝지어, 40번 부서는 아예 나오지 않는다. 하지만 "40번 부서도 (0명이라도) 보여주고 싶다"면? → **Outer Join**

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/1.png)

---

## 1. Outer Join

```sql
select *
from dept d left outer join emp e
on d.deptno = e.deptno;
```

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/2.png)

일반 조인과 달라진 점:

| 항목 | Inner Join | Outer Join |
|------|------|------|
| 표현 | `from A, B where` | `from A left outer join B on` |
| 결과 | 매칭되는 것만 (14줄) | 기준 테이블 전부 (**15줄**) |

> 💡 `left outer join`은 **왼쪽 테이블(dept)을 기준**으로, 매칭 안 되는 것도 전부 보여준다. 그래서 매칭되는 emp가 없는 **40번 Operations 부서도 나온다**(emp 쪽은 NULL). 조건에 `where`가 아니라 **`on`**을 쓴다.

**요약 — 부서별 인원수:**

```sql
select d.deptno, dname, count(e.deptno)
from dept d left outer join emp e
on d.deptno = e.deptno
group by d.deptno, dname
order by d.deptno asc;
```

![Desktop View](/assets/img/Database/SQL/Outer-Join-Transaction/3.png)

> 💡 `count(e.deptno)`로 부서별 인원을 세면, 40번 부서는 **0명**으로 나온다. `group by`로 묶었으니 `select`에는 그룹화한 컬럼과 집계함수만 올 수 있다.

---

## 2. 트랜잭션 (Transaction)

> **트랜잭션이란?** DML 작업 시 **세부 업무가 모두 성공해야 전체를 성공으로 간주**하는 논리적 업무 수행 단위.

```
DML 작업 → (임시 상태) ──commit──► 확정
                       └──rollback──► 되돌리기
```

| 명령 | 역할 |
|------|------|
| `commit;` | 변경을 **확정**(저장) |
| `rollback;` | 변경을 **취소**(되돌리기) |

> 💡 `INSERT`/`UPDATE`/`DELETE`(DML)는 바로 저장되는 게 아니라 **`rollback`으로 되돌릴 수 있다.** 확정하려면 **`commit`**이 필요하다(체크포인트). 이 덕분에 실수를 방지할 수 있다.

> ⚠️ 반면 **DDL**(`create`, `alter`, `drop`)은 수행할 때마다 **commit이 자동으로 동반**되어 즉시 확정된다. 그래서 롤백이 안 되니 조심해야 한다.

---

## 📝 정리

```
아우터 조인 & 트랜잭션
├─ Outer Join  기준 테이블 전부 표시 (매칭 안 돼도)
│              A left outer join B on A.키=B.키
├─ 트랜잭션    DML은 commit 전까지 rollback 가능
└─ DDL         자동 commit → 롤백 불가 (주의)
```

| 개념 | 한 줄 정의 |
|------|------|
| **Inner vs Outer Join** | 매칭만 vs 기준 전부 |
| **commit / rollback** | 확정 / 되돌리기 |
| **트랜잭션** | 전부 성공해야 성공인 업무 단위 |

Outer Join은 "한쪽에 없어도 다 보여준다", 트랜잭션은 "commit 전까지 되돌릴 수 있다"가 핵심이다. DML은 롤백이 되지만 **DDL은 자동 commit**이라는 차이를 꼭 기억하자.
