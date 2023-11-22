---
title: 세션 DB연동
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

이번 시간에는 세션을 만들어서 로그인 폼을 만들어보고 JSP로 DB와 연동해보겠습니다!

<loginForm.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
 <%
 	String cust_id =(String)session.getAttribute("cust_id");
 	String cust_name =(String)session.getAttribute("cust_name");
 	Boolean login = false;
 	
 	if(cust_id !=null && cust_name !=null){
 		login = true; //로그인 상태
 	}
 
 %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<form name="loginform" method="POST" action="loginCheck.jsp">

	<table>
		<caption>로그인 폼</caption>
			<tr>
			<th>아이디</th>
			<td><input type="text" name="cust_id" size ="10" maxlength="10" required></td>
			</tr>
			
			<tr>
				<th>아이디</th>
				<td><input type="password" name="cust_pw" size ="10" maxlength="10" required></td>
			</tr>
			
			<tr>
				<td colspan ="2" style ="text-align:center;">
				
<%
	if(login){ //로그인 상태
		out.print("<input type ='submit' value='로그인' disabled>" + 
					"<input type = 'button' value='(" + cust_name + ")님 로그아웃' " +
					"onClick=location.href='./logout.jsp'></td>");
	}
	else{
		out.print("<input type='submit' value ='로그인'>" + "<input type='button' value='로그아웃' disabled></td>");
	}

%>
			</tr>
	</table>


</form>


</body>
```

로그인 폼을 만들었습니다. cust\_id와 cust\_name을 만들었습니다. getAttribute()로 세션의 아이디와 이름을 가져오는데 없으면 null이겠죠? 그래서 boolean타입의 login을 만들었습니다. false로 초기화를 했습니다. 앞으로 이 변수의 값을 이용해서 로그인의 유무를 판단합니다.

아이디와 비밀번호를 입력하고 cust\_id와 cust\_pw를 다른 jsp파일에서 조회를 할 겁니다.

그리고 로그인이 되어있으면 로그인 버튼을, 로그인이 안 됐으면 로그아웃 버튼을 비활성화 시켰습니다.

<loginCheck.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
	String user_id = "root";
	String user_pw="admin";
	String user_name="관리자";
	
	String cust_id = request.getParameter("cust_id");
	String cust_pw = request.getParameter("cust_pw");
	
	if(cust_id.equals(user_id)&& cust_pw.equals(user_pw)){
		session.setAttribute("cust_id", user_id);
		session.setAttribute("cust_name", user_name);
		
		out.print("<br>" + session.getAttribute("cust_id") + "("+session.getAttribute("cust_name")+")님 방문을 환영합니다.</br>");
	}
	else{
		out.print("회원 가입 후 방문하십시요<br>");
	}

	


%>

<a href="./loginForm.jsp">[로그인 폼]</a>

</body>
```

로그인폼에서 적은 아이디와 비밀번호를 체크하는 jsp입니다. id 와 pw를 미리 정했습니다. 나중에는 db와 연동해서 각자의 계정을 만들 수 있겠죠?

그래서 들어온 값들이 아이디와 비번이 일치하면 setAttribute()로 cust\_id와 cust\_name를 생성합니다. 그리고 아까의 로그인폼으로 다시 돌아가는 링크를 만들었습니다. 이제 로그아웃도 만들어야겠죠?

<logout.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
	session.invalidate();
	out.print("로그 아웃 하셨습니다.");
	

%>

<a href="./loginForm.jsp">[세션 삭제 확인]</a>
</body>
```

이 페이지로 넘어오면 session.invalidate()로 세션정보를 모두 삭제합니다. 그리고 다시 로그인 폼으로 가겠습니다.

![Desktop View](/assets/img/Frontend/JSP/Session-DB/1.png)

로그인폼입니다. 처음엔 로그인이 안 됐으니 로그아웃 버튼을 비활성화가 되어있네요. 그리고 알맞은 아이디와 비밀번호를 입력하면?

![Desktop View](/assets/img/Frontend/JSP/Session-DB/2.png)

이렇게 나오네요 ㅎㅎ 그리고 다시 로그인폼으로 가보겠습니다.

![Desktop View](/assets/img/Frontend/JSP/Session-DB/3.png)

이제는 로그인버튼이 비활성화되어있네요 ㅎ 이제 다시 로그아웃을 하면?

![Desktop View](/assets/img/Frontend/JSP/Session-DB/4.png)

잘나오네요 ㅎㅎ 재밌습니다.

이제 DB와 연동하겠습니다. 이번엔 MariaDB를 써보겠습니다. 먼저 DB를 다운받고 드라이버 커넥터를(jar파일) lib폴더에 넣었습니다.

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
 <%@ page import ="java.sql.*" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>JDBC 드라이버 로딩</title>
</head>
<body>
<%
	String driverClass = "org.mariadb.jdbc.Driver";

	try{
		Class.forName(driverClass);
		out.print("JDBC Driver loading 성공!");
	}catch(ClassNotFoundException err){
		out.print("JDBC Driver loading 실패 ㅜ");
	}

	//Maria DB 연결
	
	String url ="jdbc:mariadb://localhost:3307";
	String id = "root";
	String pw = "1234";
	
	Connection conn = null;
	
	try{
		conn = DriverManager.getConnection(url,id,pw);
		out.print("MariaDB 서버 연결 성공");
	}catch(SQLException sqlerr){
		out.print("MariaDB 서버 연결 실패 ㅜ"+ "<br>");
		out.print(sqlerr.getMessage());
	}finally{
		if (conn !=null){
			try{
				conn.close();
				out.print("MariaDB 서버 연결 종료!<br>");
			}catch(Exception conerr){
				conerr.printStackTrace();
			}
		}
	}
%>
</body>
```

먼저 문자열 driverClass를 만들고 "org.mariadb.jdbc.Driver"를 대입했습니다. 그리고 try 문으로 Class.forName()인자에 driverClass를 넣었습니다. 잘 되면 성공문을 출력하고 아니면 실패를 출력하게 했습니다. 이미 설정을 다 끝내놔서 잘 연동됐을겁니다!

그리고 이제 DB를 연결시켜보겠습니다. url과 id와 pw를 각각 문자열로 만들었습니다. 이것은 mariadb의 아이디와 비밀번호입니다. 그리고 Connection객체의 conn을 만들었습니다. 그리고 conn에 DriverManager.getConnection()를 사용해서 연결과 그 정보를 conn에 넣었습니다. 마찬가지로 try문에 넣어서 실패하면 실패라고 출력하게 했습니다. 그리고 finally로 conn.close()로 연결을 종료하게했습니다. 실행해볼까요?

![Desktop View](/assets/img/Frontend/JSP/Session-DB/5.png)

잘나오네요 ㅎㅎ 드라이버 연결과 서버연결 그리고 연결종료가 잘 이루어졌습니다!

![Desktop View](/assets/img/Frontend/JSP/Session-DB/6.png)

연결이 잘 되면 저렇게 data 소스탭에 생깁니다. 그리고 연동이 되었기 때문에 저기를 통해서 db를 볼 수 도 있답니다! 연결방법은 예전 JAVA포스팅에 자세하게 올려놨으니 참고하세요!

이번시간에는 여기까지하겠습니다!
