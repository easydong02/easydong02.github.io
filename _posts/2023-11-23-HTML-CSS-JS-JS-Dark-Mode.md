---
title: [Frontend] Javascript로 다크모드 만들기
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript]
render_with_liquid: false
future: true
---

이번시간엔 버튼을 만들고 그 버튼으로 야간모드와 주간모드를 만들어보겠습니다!!

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    

    <input type = "button" value = "night" onclick = 
    "document.querySelector('body').style.backgroundColor = 'black';
     document.querySelector('body').style.color = 'white';    
    ">

    <input type = "button" value = "day"
       onclick = "document.querySelector('body').style.backgroundColor = 'white';
            document.querySelector('body').style.color = 'black';  ">
  <style>
    .js {
      font-weight: bold;
      color : blue;
    }
    
    .first {
      color : green;
      
    }
    #first {
      color : yellowgreen;
    }
    span{
      color : red;
    }
  </style>
  
  <h1><a href = "index.html">web</a></h1>
  <h2> JavaScript 란 무엇인가?</h2>
  <p>
    <span class = "js">JavaScript</span>
    <span class = "first">여기는 초록색</span>amskdlamskldmaksldmaksldmaskdlmaksldmkalmskdlmakslmdkalmsdkmaksmd
    <span id = "first">홀리홀리</span>
    <span class = "js">하이하이</span>
    klamskdlmaklsmdklamskdmkasldam</p>
  
  
</body>
</html>
```

바디파트에서 input을 보겠습니다 버튼 2개를 만들고 하나는 night, 다른 하나는 day로 했습니다 ㅎㅎ

document.querySelector('body') 이것은 인자에 넣은 파트 전부에 영향을 줄 수 있는 실행문입니다. 예를 들어 지금같이 'body'를 인자로 넣으면, 그 body 전체에 대한 영향을 끼치죠! 그래서 night버튼을 눌렀을 때 전체 바디의 배경색을 검정으로, 글자는 흰색으로 하게 했습니다. day버튼은 반대겠죠? ㅎㅎ

실행해보겠습니다!

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-Night-Mode/1.png)

와우! 신기하네요 ㅎㅎ 이번엔 day버튼을 눌러보겠습니다!

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-Night-Mode/2.png)

신기하네요 ㅎㅎ

이번시간엔 여기까지 하도록 하겠습니다~