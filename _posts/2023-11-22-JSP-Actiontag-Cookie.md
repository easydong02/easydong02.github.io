---
title: "[JSP] 여러가지 액션태그와 쿠키"
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 JSP의 **파라미터 전달**, **액션 태그(include·redirect)**, 그리고 **쿠키(Cookie)**를 정리한다.

---

## 1. 폼 데이터 전달 — getParameter

폼에서 `join.jsp`로 데이터를 넘긴다. **단일 값은 `getParameter`, 복수 값(체크박스)은 `getParameterValues`**.

```jsp
아이디: <%= request.getParameter("id") %>
취미:
<%
    String[] hobby = request.getParameterValues("hobby");   // 복수 → 배열
    if (hobby != null) {
        for (int i = 0; i < hobby.length; i++) out.print(hobby[i] + " ");
    } else {
        out.print("취미를 선택하지 않았습니다.");
    }
%>
```

| 메소드 | 대상 |
|------|------|
| `getParameter("name")` | 단일 값 |
| `getParameterValues("name")` | 복수 값(체크박스 등) → 배열 |

---

## 2. 액션 태그 — jsp:include

`<jsp:include>`로 다른 JSP를 포함하며 **파라미터를 넘길** 수 있다.

```jsp
<jsp:include page="./include/inc_act_multi.jsp">
    <jsp:param name="para1" value="p1값"/>
    <jsp:param name="para3" value="파라미터3값1"/>
    <jsp:param name="para3" value="파라미터3값2"/>   <!-- 같은 이름 여러 개 -->
</jsp:include>
```

포함된 파일에서는 넘어온 파라미터를 받는다.

```jsp
String[] para3 = request.getParameterValues("para3");   // 복수
<%= request.getParameter("para1") %>                     <!-- 단수 -->
```

---

## 3. redirect vs include

```jsp
// redirect: 다른 페이지로 '이동' (? 뒤는 넘길 파라미터)
String encode_para = URLEncoder.encode("대한민국", "utf-8");   // 한글 인코딩
response.sendRedirect("redirect_check.jsp?para=" + encode_para);
```

```jsp
// include: 다른 페이지를 '가져와 붙임' (이동 X)
<%@ include file="./include/inc_header.jsp" %>
<%@ include file="./include/sum.jsp" %>
<%@ include file="./include/inc_bottom.jsp" %>
```

| 방식 | 동작 |
|------|------|
| **`sendRedirect`** | 다른 페이지로 **이동** (URL 바뀜) |
| **`include`** | 다른 페이지를 **가져와 합침** (머리말/꼬리말) |

> 💡 `include`는 사이트의 **고정 머리말·꼬리말**을 재사용할 때 유용하다. `redirect`는 실제로 페이지를 이동시키며, `?` 뒤에 파라미터를 실어 보낼 수 있다(한글은 `URLEncoder`로 인코딩).

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/6.png)

---

## 4. 쿠키 (Cookie)

> **쿠키란?** HTTP의 **비연결·비상태** 문제를 해결하는 방법 중 하나. 클라이언트에 저장되어, 유효시간 이전에는 브라우저를 닫아도 계속 전송된다.

**생성:**

```jsp
Cookie cookie = new Cookie("id", "pass");
cookie.setMaxAge(300);          // 유효시간 300초(5분)
response.addCookie(cookie);     // 응답에 쿠키 추가
```

**조회:**

```jsp
Cookie[] cookies = request.getCookies();
for (Cookie c : cookies) {
    out.print("쿠키 이름: " + c.getName() + ", 값: " + c.getValue());
}
```

**삭제** — 유효시간을 0으로:

```jsp
for (Cookie c : cookies) {
    c.setMaxAge(0);             // 0초 → 즉시 삭제
    response.addCookie(c);
}
```

| 동작 | 방법 |
|------|------|
| 생성 | `new Cookie` + `setMaxAge` + `addCookie` |
| 조회 | `request.getCookies()` |
| 삭제 | `setMaxAge(0)` |

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/8.png)

---

## 📝 정리

```
액션 태그 & 쿠키
├─ 파라미터  getParameter(단일) / getParameterValues(복수)
├─ include   페이지를 가져와 합침 (머리말/꼬리말)
├─ redirect  다른 페이지로 이동 (?로 파라미터)
└─ 쿠키       setMaxAge로 유효시간(0이면 삭제)
```

| 개념 | 한 줄 정의 |
|------|------|
| **getParameterValues** | 복수 값을 배열로 |
| **include vs redirect** | 합침 vs 이동 |
| **Cookie** | 클라이언트에 저장되는 상태 |

`include`와 `redirect`는 "합치기 vs 이동하기"로 구분되고, 쿠키는 "클라이언트에 상태를 저장"한다. 이 세 가지로 여러 JSP 페이지를 유기적으로 연결하는 웹 흐름을 만들 수 있다.
