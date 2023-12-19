---
title: "[JSP] 여러가지 액션태그와 쿠키"
date: 2023-11-22 00:00:00 +0900
categories: [Frontend, JSP]
tags: [jsp]
render_with_liquid: false
future: true
---

JSP로 돌아왔습니다! 오늘은 액션태그로 웹간 이동을 해보겠습니다!

간단한 회원가입 폼을 만들고 그걸을 다른 jsp파일로 데이터를 옮겨보겠습니다.

**<foam.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
<% request.setCharacterEncoding("UTF-8"); %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>회원 가입 폼</title>
</head>
<body>
	<form name = "customer_form" method ="post" action="join.jsp">
		<table>
			<caption>회원 가입</caption>
			<tr>
				<th><span class="msg_red">*</span>아이디</th>
				<td><input type="text" name="id" size="10" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>비밀번호</th>
				<td><input type="password" name="pw" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>이름</th>
				<td><input type="text" name="name" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>성별</th>
				<td><input type="radio" name="gender" value="M">남자&nbsp;
				<input type="radio" name="gender" value="F">여자
				</td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>취미</th>
				<td><input type="checkbox" name="hobby" value="등산">등산&nbsp;&nbsp;&nbsp;&nbsp;
				<input type="checkbox" name="hobby" value="독서">독서&nbsp;&nbsp;&nbsp;&nbsp;
				<input type="checkbox" name="hobby" value="음악듣기">음악듣기&nbsp;&nbsp;&nbsp;&nbsp;
				<input type="checkbox" name="hobby" value="영화보기">영화보기</td>
			</tr>
			<tr>
				<td colspan="2" style="text-align:center;">
					<input type="submit" value="전송">
					<input type="reset" value="취소">
				</td>
			
			</tr>
		
		</table>
```

길어보이지만 대부분 HTML문법입니다. 요약하자면 아이디와 비밀번호 이름, 성별, 취미를 작성하고 전송버튼을 누르면 join.sjp로 정보를 가지고 갑니다.

**<join.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>


<% request.setCharacterEncoding("UTF-8"); %>
    
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<% %>


아이디: <%=request.getParameter("id") %>
비밀번호: <%=request.getParameter("pw") %>
이름: <%=request.getParameter("name") %>
성별: <%=request.getParameter("gender") %>
취미: 

<%
String[] hobby = request.getParameterValues("hobby");


if(hobby != null){
	for(int i=0; i<hobby.length; i++){
		out.print(hobby[i]+" ");
	}
}
else{
	out.print("취미를 선택하지 않았습니다.");
}
```

join.jsp입니다. request.getParameter()로 각각의 정보를 가져왔습니다. 다만 취미를 복수 선택이 가능하기에 문자열 배열로 지정한 뒤 for문으로 전부 가져왔습니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/1.png)

실행한 뒤 값을 입력하고 전송버튼을 눌렀습니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/2.png)

잘 나오네요 ㅎㅎ 최근에 등산을 시작해서 매주 주말마다 가는데 정말 체력증진에 좋은 것같습니다 ㅎㅎ 최근에 비교적 가까운 계양산을 가봤는데 좋더라고요! 뭐 시간이 난다면 한번 포스팅해보겠습니다. 이번주말은 백신2차 맞아서 못갈듯..

이제 멀티 액션태그를 알아봅시다.

**<actionmulti.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<% request.setCharacterEncoding("UTF-8"); %>    
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>


<jsp:include page="./include/inc_act_multi.jsp">
	<jsp:param name="para1" value="p1값"/>
	<jsp:param name="para2" value="p2값"/>
	<jsp:param name="para3" value="파라미터3값1"/>
	<jsp:param name="para3" value="파라미터3값2"/>
	<jsp:param name="para3" value="파라미터3값3"/>
	<jsp:param name="para3" value="파라미터3값4"/>
</jsp:include>

```

보통 jsp: include는 include폴더를 만들어서 거기엔 include파일만 넣습니다.

여기선 para데이터를 1부터 3까지 넣고 3은 같은 name으로 다른 value 4개를 만들었습니다. page는 include가 있는 파일 경로를 적었습니다.

**<inc\_act\_multi.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
<% request.setCharacterEncoding("UTF-8"); %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
	String[] para3 =request.getParameterValues("para3");
	
if(para3 != null){
	for(int i =0; i<para3.length;i++){
		out.print("전송파라미터3: " + para3[i]+"<br>");
	}
}

%>

전송 파라미터 1: <%=request.getParameter("para1") %><br>
전송 파라미터 2: <%=request.getParameter("para2") %><br>

```

para3만 문자열 배열로 만들었습니다. 복수 데이터는 getParameterValues로 합니다 단수 데이터들은 getParameter()로 하구요. 그래서 para1,2는 저렇게 썼습니다

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/3.png)

actionmulti.jsp 에서 바로 inc\_act\_multi.jsp로 간 것입니다. 따라서 첫번째 jsp파일에 있는 값들을 가져와서 출력을 했죠 ㅎㅎ

이번엔 3개를 연결해보겠습니다.

**<redirect.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
    
<% request.setCharacterEncoding("UTF-8"); %>



<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<form name = "form" method ="post" action="join_redirect.jsp">
		<table>
			<caption>redirect</caption>
			<tr>
				<th><span class="msg_red">*</span>아이디</th>
				<td><input type="text" name="id" size="10" maxlength="10" required></td>
			</tr>
			<tr>
				<th><span class="msg_red">*</span>비밀번호</th>
				<td><input type="password" name="pw" size="11" maxlength="10" required></td>
			</tr>
			<tr>
				<td colspan="2" style="text-align:center;">
					<input type="submit" value="전송">
					<input type="reset" value="취소">
				</td>
			</tr>
		</table>
	</form>

	...생략
```

아이디와 비밀번호만 쓰고 foam처럼 join\_redirect.jsp로 이동하게 했습니다.

**<join\_redirect.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
<%@ page import = "java.net.URLEncoder" %>

<% request.setCharacterEncoding("UTF-8"); %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
	String cust_id= request.getParameter("id");
	String cust_pw= request.getParameter("pw");

	out.print(cust_id + "<br>");
	out.print(cust_pw);
	
	String para = "대한민국";
	String encode_para = URLEncoder.encode(para,"utf-8");
	
	
	//response.sendRedirect("redirect.jsp");
	response.sendRedirect("redirect_check.jsp?para=" + encode_para); //? 뒷부분은 옵션임. 그 페이지로 가는데 옵션을 들고가는 것임.

%>

	<p><a href="redirect.jsp">[폼으로 돌악가기]</a></p>

```

여기선 일단 넘겨받은 id 와 pw를 출력하고 para변수를 만들어 대한민국을 넣었습니다. 그리고 encode\_para를 만들어서 URLENcoder.encode()에 para를 인자로 넣고 그 정보를 변수에 넣었습니다. 그리고 reponse.sendRedirect()를 사용하여 그 인자에

"redirect\_check.jsp?para=" + encode\_para 를 넣었는데 이것은 redirect\_check.jsp로 이동하는데 물음표 뒷부분 para = encode\_para를 갖고 가라는 거죠.

**<redirect\_check.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<% request.setCharacterEncoding("UTF-8"); %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>


<%
String para= request.getParameter("para");

out.print("파라미터 = "+para);
%>
```

마지막 jsp파일입니다. 전 파일에서 가져온 para를 출력합니다. 이제 redirect파일을 실행해봅시다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/4.png)

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/5.png)

두번째 파일을 거쳐 마지막 파일이 나왔네요 ㅎㅎ

```
<%@ include file = "./include/inc_header.jsp" %>

<%@ include file = "./include/sum.jsp" %>

<%@ include file = "./include/inc_bottom.jsp" %>
```

이건 뭘까요? include file은 그 페이지를 자기 페이지로 가져오는 것입니다. 그 주소로 이동하는 것과는 차이가 있죠?

미리 만든 헤더파일과 바텀 파일을 만들었습니다. 그리고 1부터 10까지 정수의 합을 넣은 sum.jsp을 그 사이에 넣었습니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/6.png)

헤더영역입니다는 헤더파일에, bottom영역입니다. 는 바텀파일에 있었죠. 그리고 가운데는 sum.jsp파일이구요. 이런식으로 파일을 가져와서 갖다 붙힐 수 있습니다. 보통 사이트에서 머리말 꼬리말처럼 고정적으로 띄워져 있는 것과 같습니다.

**쿠키**

쿠키는 비연결 비상태유지에 대한 문제점을 해결하는 방법 중 하나입니다. 쿠키를 사용하여 클라이언트의 정보를 웹서버로 전송할 수 있습니다. 웹서버는 전송받은 정보를 확인해서 이전과 동일인인지 여부를 확인할 수 있습니다. 생성된 쿠키는 클라이언트에 저장되어 삭제되기 전 또는 브라우저를 종료하더라도 유효시간 이전에는 계속해서 전송됩니다.

**<cookieExam.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<% request.setCharacterEncoding("UTF-8"); %>

<%@ include file ="./include/inc_header.jsp" %>
<%
	Cookie cookie = new Cookie("id","pass");
	cookie.setMaxAge(300); //초단위(5분)
	response.addCookie(cookie);
	
	
	out.print("쿠키 생성 성공");
%>

쿠키 이름: <%=cookie.getName() %><br>
쿠키 값: <%=cookie.getValue() %><br>
유효시간: <%=cookie.getMaxAge() %> 초<br>

<p><a href ="cookieDelete.jsp">[쿠키 삭제]</a>

<%@ include file ="./include/inc_bottom.jsp" %>
```

Cookie 객체 cookie를 만들어서 쿠키 이름은 id 값은 pass로 했습니다. 쿠키의 유효시간은 setMaxAge()로 300초로 했습니다. 그 다음 response.addCookie()로 그 쿠키를 추가했습니다. 그다음 쿠키 삭제 버튼을 만들어서 쿠키삭제 파일로 이동하게 했습니다.

**<cookieDelete.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<% request.setCharacterEncoding("UTF-8"); %>

<%@ include file ="./include/inc_header.jsp" %>

<%
	Cookie[] cookies=request.getCookies();

	if(cookies != null){
		for( int i =0; i<cookies.length; i++){
			cookies[i].setMaxAge(0);
			response.addCookie(cookies[i]);
			out.print("쿠키 정보가 삭제되었습니다.");
		}
	}
	else{
		out.print("설정된 쿠키 정보가 없습니다.");
	}
%>
<p><a href ="cookieCheck.jsp">[쿠키 확인]</a>

<%@ include file ="./include/inc_bottom.jsp" %>
```

여기선 getCookies()로 저장된 쿠키를 다 가져왔습니다. 그리고 for문으로 쿠키 유효시간들을 다 0초로 만들었습니다. 사실상 없애는거죠. 그리고 버튼을 만들어서 쿠키확인을 하도록 하겠습니다.

**<cookieCheck.jsp>**

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<% request.setCharacterEncoding("UTF-8"); %>

<%@ include file ="./include/inc_header.jsp" %>

<%
	Cookie[] cookies=request.getCookies();

	if(cookies != null){
		for( int i =0; i<cookies.length; i++){
			String name = cookies[i].getName();
			String value = cookies[i].getValue();
			
			out.print("쿠키 이름: " + name + "<br>");
			out.print("쿠키 밸류: " + value + "<br>");
		}
	}
	else{
		out.print("설정된 쿠키 정보가 없습니다.");
	}
%>
<p><a href ="cookieDelete.jsp">[쿠키 삭제]</a>

<%@ include file ="./include/inc_bottom.jsp" %>
```

있는 쿠키들은 getName()과 getValue()로 다 가져옵니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/7.png)

먼저 cookieExam으로 쿠키를 만들었습니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/8.png)

쿠키삭제로 들어가서 쿠키가 삭제를 시켰습니다.

![Desktop View](/assets/img/Frontend/JSP/actiontag-cookie/9.png)

쿠키확인파트입니다. 저건 다른 세션이 켜져있어서 만들어진 겁니다. 우리가 만든 id 쿠키는 사라졌죠? ㅎㅎ 재밌습니다!

이번 시간에는 여기까지 하겠습니다!!