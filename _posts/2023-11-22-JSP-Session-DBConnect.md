---
title: "[JSP] 세션 DB연동"
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **세션(session)**으로 로그인 폼을 만들고, JSP를 **MariaDB와 연동**한다.

---

## 1. 세션 로그인 흐름

세 개의 JSP로 로그인 상태를 관리한다.

```
loginForm.jsp ──POST──► loginCheck.jsp (검증 → session에 저장)
     ▲                          │
     └──────────────────────────┘
logout.jsp (session.invalidate)
```

### loginForm.jsp — 상태에 따라 버튼 토글

```jsp
<%
    String cust_id = (String) session.getAttribute("cust_id");
    String cust_name = (String) session.getAttribute("cust_name");
    Boolean login = (cust_id != null && cust_name != null);   // 로그인 여부 판단
%>
<%
    if (login) {   // 로그인 상태 → 로그인 버튼 비활성, 로그아웃 활성
        out.print("<input type='submit' value='로그인' disabled>"
            + "<input type='button' value='(" + cust_name + ")님 로그아웃' "
            + "onClick=location.href='./logout.jsp'>");
    } else {       // 미로그인 → 반대
        out.print("<input type='submit' value='로그인'>"
            + "<input type='button' value='로그아웃' disabled>");
    }
%>
```

### loginCheck.jsp — 세션에 저장

```jsp
<%
    String cust_id = request.getParameter("cust_id");
    String cust_pw = request.getParameter("cust_pw");

    if (cust_id.equals("root") && cust_pw.equals("admin")) {
        session.setAttribute("cust_id", cust_id);       // ★ 세션에 저장
        session.setAttribute("cust_name", "관리자");
        out.print(cust_id + "님 방문을 환영합니다.");
    } else {
        out.print("회원 가입 후 방문하십시오");
    }
%>
```

### logout.jsp — 세션 삭제

```jsp
<% session.invalidate(); %>   <!-- 모든 세션 정보 삭제 -->
```

| 메소드 | 역할 |
|------|------|
| `session.getAttribute` | 세션 값 조회 |
| `session.setAttribute` | 세션 값 저장 |
| `session.invalidate` | 세션 전체 삭제 |

![Desktop View](/assets/img/Frontend/JSP/Session-DB/2.png)

---

## 2. DB 연동 (MariaDB + JDBC)

DB 드라이버 커넥터(jar)를 `lib` 폴더에 넣고, JDBC로 연결한다.

```jsp
<%@ page import="java.sql.*" %>
<%
    // ① 드라이버 로드
    Class.forName("org.mariadb.jdbc.Driver");
    out.print("JDBC Driver loading 성공!");

    // ② 접속
    Connection conn = null;
    try {
        conn = DriverManager.getConnection("jdbc:mariadb://localhost:3307", "root", "1234");
        out.print("MariaDB 서버 연결 성공");
    } catch (SQLException e) {
        out.print("MariaDB 서버 연결 실패: " + e.getMessage());
    } finally {
        if (conn != null) conn.close();   // ③ 해제
    }
%>
```

![Desktop View](/assets/img/Frontend/JSP/Session-DB/5.png)

> 💡 흐름은 자바 JDBC와 동일하다: **① 드라이버 로드(`Class.forName`) → ② 접속(`getConnection`) → ③ 해제(`close`)**. 각 단계를 `try~catch`로 감싸 성공/실패를 출력한다.

---

## 📝 정리

```
JSP 세션 & DB
├─ 세션    getAttribute/setAttribute/invalidate
├─ 로그인  상태에 따라 버튼 토글
└─ DB      Class.forName → getConnection → close
```

| 개념 | 한 줄 정의 |
|------|------|
| **session** | 사용자별 상태 유지 |
| **invalidate** | 세션 삭제(로그아웃) |
| **JDBC 연결** | 드라이버 로드 → 접속 → 해제 |

세션은 "로그인 상태를 서버가 기억"하게 해준다. `setAttribute`로 저장하고 `invalidate`로 지우는 흐름, 그리고 JDBC 3단계 연결만 잡으면 동적인 로그인 시스템을 만들 수 있다.
