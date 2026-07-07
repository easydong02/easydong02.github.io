---
title: "[JSP] 동적 구현(DB)"
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 JSP로 **DB 생성 → 테이블 생성 → 폼으로 행 삽입 → 조회 출력**의 전체 흐름을 학생 등록 예제로 구현한다.

```
dbcreate.jsp (DB 생성) → createTable.jsp (테이블)
→ formstudent.jsp (입력 폼) → insertStudent.jsp (INSERT)
→ formStudentTable.jsp (SELECT 출력)
```

---

## 1. DB·테이블 생성 (DDL)

연결 후 SQL을 `PreparedStatement.executeUpdate()`로 실행한다.

```jsp
// DB 생성
String sql = "CREATE DATABASE univ2";
pstmt = conn.prepareStatement(sql);
pstmt.executeUpdate();

// 테이블 생성 (url 끝에 /univ2 추가)
String sql = "CREATE TABLE student("
    + "studentNo int not null, name varchar(5), year tinyint,"
    + "dept varchar(10), addr varchar(50), primary key(studentNo))";
pstmt.executeUpdate();
```

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/2.png)

---

## 2. 폼 입력 → INSERT

폼에서 받은 값을 **바인딩 변수(`?`)**로 안전하게 삽입한다.

```jsp
<%
    int studentNo = Integer.parseInt(request.getParameter("studentNo"));
    String name = request.getParameter("name");
    // ... 나머지 파라미터

    String sql = "INSERT INTO student(studentNo,name,year,dept,addr) VALUES(?,?,?,?,?)";
    pstmt = conn.prepareStatement(sql);
    pstmt.setInt(1, studentNo);      // ? 자리에 값 바인딩
    pstmt.setString(2, name);
    // ...
    pstmt.executeUpdate();
%>
```

> 💡 값이 들어가는 SQL은 **`?`(바인딩 변수)**를 쓰고 `setInt`/`setString`으로 채운다. SQL 인젝션을 막고 가독성도 좋다.

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/4.png)

---

## 3. SELECT → 테이블 출력

`executeQuery()`의 결과를 **`ResultSet`**으로 받아 `while(rset.next())`로 순회한다.

```jsp
<%
    String sql = "SELECT * FROM student";
    pstmt = conn.prepareStatement(sql);
    ResultSet rset = pstmt.executeQuery();   // 조회 → ResultSet
%>
<table class="table table-dark table-striped">
    <tr><th>학번</th><th>이름</th><th>학년</th><th>전공</th><th>주소</th></tr>
<%
    while (rset.next()) {   // 다음 행이 있으면 true
%>
    <tr>
        <td><%= rset.getString("studentNo") %></td>
        <td><%= rset.getString("name") %></td>
        <td><%= rset.getString("year") %></td>
        <!-- ... -->
    </tr>
<%
    }
%>
</table>
```

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/5.png)

> 💡 `ResultSet`의 커서는 처음에 **0번째 행**(빈 곳)에 있어, `next()`를 한 번 해야 첫 행으로 간다. `next()`는 행이 있으면 true를 반환하므로 `while` 조건으로 딱 맞다. (색상은 Bootstrap으로 꾸몄다)

| 메소드 | 역할 |
|------|------|
| `executeUpdate()` | INSERT/CREATE 등 (행 수) |
| `executeQuery()` | SELECT → ResultSet |
| `rset.next()` | 다음 행으로 (있으면 true) |
| `rset.getString("컬럼")` | 현재 행의 값 |

---

## 📝 정리

```
JSP 동적 DB
├─ DDL      CREATE → executeUpdate
├─ INSERT   ? 바인딩 + setInt/setString → executeUpdate
├─ SELECT   executeQuery → ResultSet
└─ 순회     while(rset.next()) + getString
```

| 개념 | 한 줄 정의 |
|------|------|
| **PreparedStatement** | 바인딩 변수로 안전한 SQL |
| **executeUpdate/Query** | 변경 / 조회 |
| **ResultSet** | 조회 결과 (커서로 순회) |

JSP로 DB를 다루면 "폼 입력 → DB 저장 → 조회 출력"이라는 웹 애플리케이션의 핵심 흐름을 직접 구현할 수 있다. 특히 INSERT는 `?` 바인딩, SELECT는 `ResultSet` 순회가 기본 패턴이다.
