---
title: 동적 구현(DB)
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

이번 시간에는 jsp로 DB에 여러가지를 해보겠습니다! 여기서는 학생등록 폼을 만들어서 그것을 DB에 넣고 행도 삽입하고 확인해보겠습니다!

<dbcreate.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%@ page import = "java.sql.*" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<% 
	String driverClass = "org.mariadb.jdbc.Driver";

	try{
		Class.forName(driverClass);
		out.println("JDBC 드라이버 로딩 성공"+"<br>");
	}
	catch(ClassNotFoundException e){
		out.print("JDBC드라이버 로딩 실패"+"<br>");
	}
	String url = "jdbc:mariadb://localhost:3307";
	String id = "root";
	String pw = "1234";
	
	Connection conn = null;
	PreparedStatement pstmt = null;
	
	try{
		conn = DriverManager.getConnection(url,id,pw);
		out.print("MariaDB 서버 연결 성공"+"<br>");
		
		//SQL 처리
		
		String sql ="CREATE DATABASE univ2";
		pstmt = conn.prepareStatement(sql);
		pstmt.executeUpdate();
		out.print("데이터베이스 생성 성공"+"<br>");
	}
	catch(SQLException e){
		out.print("MariaDB  실패."+"<br>");
	}
	finally{
		//데이터 베이스 연결 종료
		if(pstmt != null){
			try{
				pstmt.close();
				out.print("pstmt 연결 종료" +"<br>");
			}catch(Exception conerr){
				conerr.printStackTrace();
			}
		}
		if(conn != null){
			try{
				conn.close();
				out.print("MariaDB 서버 연결 종료" +"<br>");
			}catch(Exception conerr){
				conerr.printStackTrace();
			}
		}
	}
%>
</body>
</html>
```

먼저 데이터베이스를 만들어야겠죠? 드라이버를 연결하고 DB에 접속한 뒤 문자열 sql에 sql문을 입력했습니다. 그리고 PreparedStatement 객체인 pstmt를 만들어서 거기에 연결 정보를 넣고 이 변수를 사용해서 executeUpdate()로 저 sql문을 실행했습니다. 이제 univ2가 만들어졌을까요?

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/1.png)

잘 만들어졌네요 ㅎㅎ 이제 테이블을 만들어볼까요?

<createTable.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%@ page import = "java.sql.*" %>


<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<% 
	String driverClass = "org.mariadb.jdbc.Driver";
	

	try{
		Class.forName(driverClass);
		out.println("JDBC 드라이버 로딩 성공"+"<br>");
	}
	catch(ClassNotFoundException e){
		out.print("JDBC드라이버 로딩 실패"+"<br>");
		
	}
	
	String url = "jdbc:mariadb://localhost:3307/univ2";
	String id = "root";
	String pw = "1234";
	
	Connection conn = null;
	PreparedStatement pstmt = null;
	
	try{
		conn = DriverManager.getConnection(url,id,pw);
		out.print("MariaDB 서버 연결 성공"+"<br>");
		
		//SQL 처리
		String sql ="CREATE TABLE student("
					+"studentNo		int		not null,"
					+"name	varchar(5),"
					+"year		tinyint,"
					+"dept		varchar(10),"
					+"addr		varchar(50),"
					+"primary	key(studentNo))";

		pstmt = conn.prepareStatement(sql);
		pstmt.executeUpdate();
		out.print("테이블 생성 성공"+"<br>");
	}
	catch(SQLException e){
		out.print("MariaDB  실패."+"<br>");
	}
	finally{
		//데이터 베이스 연결 종료
		if(pstmt != null){
			try{
				pstmt.close();
				out.print("pstmt 연결 종료" +"<br>");
			}catch(Exception conerr){
				conerr.printStackTrace();
			}
		}
		if(conn != null){
			try{
				conn.close();
				out.print("MariaDB 서버 연결 종료" +"<br>");
			}catch(Exception conerr){
				conerr.printStackTrace();
			}
		}
	}

%>

</body>
</html>
```

이번엔 테이블을 만들어봅시다. 연결까지는 같은 로직 전개입니다~ 그리고 다른 점이라면 sql에 테이블을 만드는 sql문을 넣었습니다 ㅎㅎ 실행시켜볼까요?

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/2.png)

잘 생성됐습니다 ㅎㅎ cmd로 하다가 자바로 하니까 또 새롭네용~ㅎㅎ

이제 여기에 삽입할 행을 하나씩 만듭시다. 그건 이제 폼을 만들어서 전송하는걸로!

<formstudent.jsp>

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
<% request.setCharacterEncoding("UTF-8"); %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>학생 등록 폼</title>
</head>
<body>
	<form name = "customer_form" method ="post" action="insertStudent.jsp">
		<table>
			<caption>학생 정보 입력</caption>
			<tr>
				<th><span class="msg_red">*</span>학번</th>
				<td><input type="text" name="studentNo" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>이름</th>
				<td><input type="text" name="name" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>학년</th>
				<td><input type="radio" name="year" value="1">1학년&nbsp;
				<input type="radio" name="year" value="2">2학년&nbsp;
				<input type="radio" name="year" value="3">3학년&nbsp;
				<input type="radio" name="year" value="4">4학년
				</td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>전공</th>
				<td><input type="text" name="dept" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>주소</th>
				<td><input type="text" name="addr" size="30" maxlength="10" required></td>
			</tr>
			<tr>
				<td colspan="2" style="text-align:center;">
					<input type="submit" value="전송">
					<input type="reset" value="취소">
				</td>
			
			</tr>
		
		</table>
	</form>
</body>
</html>
```

저번에 쓰던 폼을 가져와서 살짝 수정했습니다. 실행해보죠

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/3.png)

폼이 잘 나오네요 ㅎㅎ 여기에 제 정보를 입력했습니다 ㅋㅋ 화석학번 인증.. 그런데 폼만있으면 뭐합니까? 이 정보로 db를 보내야겠죠?

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import = "java.sql.*" %>
<% request.setCharacterEncoding("UTF-8"); %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
int cust_studentNo= Integer.parseInt(request.getParameter("studentNo"));
String cust_name= request.getParameter("name");
int cust_year= Integer.parseInt(request.getParameter("year"));
String cust_dept= request.getParameter("dept");
String cust_addr= request.getParameter("addr");


String driverClass = "org.mariadb.jdbc.Driver";


try{
	Class.forName(driverClass);
	out.println("JDBC 드라이버 로딩 성공"+"<br>");
}
catch(ClassNotFoundException e){
	out.print("JDBC드라이버 로딩 실패"+"<br>");
	
}

String url = "jdbc:mariadb://localhost:3307/univ2";
String id = "root";
String pw = "1234";

Connection conn = null;
PreparedStatement pstmt = null;

try{
	conn = DriverManager.getConnection(url,id,pw);
	out.print("MariaDB 서버 연결 성공"+"<br>");
	
	//SQL 처리
	String sql ="INSERT INTO student(studentNo,name,year,dept,addr) VALUES(?,?,?,?,?)";
	
	pstmt = conn.prepareStatement(sql);
	pstmt.setInt(1,cust_studentNo);
	pstmt.setString(2,cust_name);
	pstmt.setInt(3,cust_year);
	pstmt.setString(4,cust_dept);
	pstmt.setString(5,cust_addr);
	
	pstmt.executeUpdate();
	out.print("행삽입 성공"+"<br>");	
}
catch(SQLException e){
	out.print("MariaDB  실패."+"<br>");
}
finally{
	//데이터 베이스 연결 종료
	if(pstmt != null){
		try{
			pstmt.close();
			out.print("pstmt 연결 종료" +"<br>");
		}catch(Exception conerr){
			conerr.printStackTrace();
		}
	}
	if(conn != null){
		try{
			conn.close();
			out.print("MariaDB 서버 연결 종료" +"<br>");
		}catch(Exception conerr){
			conerr.printStackTrace();
		}
	}
}
%>
</body>
</html>
```

이게 그 jsp파일입니다. 폼에서 가져온 데이터를 바탕으로 DB에 우리가 만든 테이블에 행을 삽입해볼게요!

여기서 눈여겨 볼것은 다른 sql문과 달리 변수가 들어가 있는 문자열이라 바로 ?문자를 사용했습니다. 그리고 pstmt.setString과 Int로 각각의 ?문자에 받아온 변수들을 넣고 sql문을 실행시켰습니다.

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/4.png)

잘 들어갔네요 ㅎㅎ 심심해서 하나 더 넣어봤습니다! 이제 보니까 학번부분에 콤마가 들어갔네요~ 나중에 수정할게요 ㅎㅎ

이제 다 끝났지만 들어간 데이터를 자바로 확인까지 해보겠습니다!

<formStudentTable.jsp>

```
<html>
<head>
<meta charset="UTF-8">
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
<title>Insert title here</title>
</head>
<body>
<%

Connection conn =null;			//커넥션
PreparedStatement pstmt =null;	//입력
ResultSet rset=null;			//출력

String driverClass = "org.mariadb.jdbc.Driver";

try{
	Class.forName(driverClass);
}catch(ClassNotFoundException e){
	out.print("JDBC 드라이버 연결 실패");
	
}

String url = "jdbc:mariadb://localhost:3307/univ2";
String id="root";
String pw="1234";

try{
	conn = DriverManager.getConnection(url,id,pw);
	
	
	//SQL..
	String sql = "SELECT * FROM student";
	pstmt =conn.prepareStatement(sql);
	rset=pstmt.executeQuery();
%>

<table class="table table-dark table-striped">
	<caption>학생 정보 검색</caption>
	<tr>
	<th>학번</th>
	<th>이름</th>
	<th>학년</th>
	<th>전공</th>
	<th>주소</th>
	</tr>
<% 
	while(rset.next()){
		String studentNo =rset.getString("studentNo");
		String name =rset.getString("name");
		String year =rset.getString("year");
		String dept =rset.getString("dept");
		String addr =rset.getString("addr");
		
%>
		<tr>
		<td><%= studentNo%></td>
		<td><%= name%></td>
		<td><%= year%></td>
		<td><%= dept%></td>
		<td><%= addr%></td>
		</tr>

<%
	}
}catch(Exception e){
	out.print("Maria DB 실패");
}finally{
	//데이터 베이스 연결 종료
	if(pstmt != null){
		try{
			pstmt.close();
		}catch(Exception conerr){
			conerr.printStackTrace();
		}
	}
	if(conn != null){
		try{
			conn.close();
		}catch(Exception conerr){
			conerr.printStackTrace();
		}
	}
}
%>
</table>
</body>
</html>
```

DB에 연결까지는 다 똑같습니다 ㅎㅎ 여기에 table을 만든 뒤 이번엔 select문을 자바로 쓰겠습니다. 보통은 pstmt까지 썼는데 이번엔 한번 더 씁니다. 바로 pstmt.executeQuery()의 정보를 갖는 변수 rset을 만들었습니다. 그리고 wihle문을 주목해주세요! rset.next()가 보이네요 ㅎㅎ 이것은 행을 위에서부터 한줄 내려오는 것입니다. 그리고 초기값은 0번째행, 즉 아무것도 없습니다. 그래서 next()로 한번 내려와야 첫 행에 가는 거죠. 그리고 next()는 값이 있으면 True고 없으면 False를 반환합니다 .그래서 조건문에 넣을 수 있죠. 그래서 로직은 한칸 내려가고 그 행의 모든 정보를 변수에 대입하고 출력하는 겁니다. 실행해볼까요?

![Desktop View](/assets/img/Frontend/JSP/DynamicDB/5.png)

awesome!!!!!!정말 신기하네요 ㅎㅎ 색깔은 부트스트랩으로 가서 그냥 모양만 좀 바꿨습니다~

이번시간엔 여기까지 하겠습니당~