---
title: "[Frontend] CSS style"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, style]
render_with_liquid: false
future: true
---

오늘은 HTML을 대충 마치고 CSS 살짝 배웠습니다. 편하더라고용 ㅎㅎ

```
<!DOCTYPE HTML>
<html>



<head>
    <meta charset="utf-8">
    <title>W3C</title>
    <style>
        a{
            color: red;
            text-decoration: none;
        }


    </style>
</head>


<body>
    <img src="https://img1.daumcdn.net/thumb/R720x0/?fname=http%3A%2F%2Ft1.daumcdn.net%2Fliveboard%2Ffaibet%2Fe0f37522d0694c1b8cb2b7a1953b6929.jpg" width="20%">
<ol>
    <h1>WEB</font></h1>
    <li><a href="1.html">HTML</a></li><br>
    <li><a href="2.html" style="color :blue; text-decoration: underline;">CSS</a></li><br>
    <li><a href="3.html">JS</a></li><br>
</ol>

<h2>CSS란 무엇인가?</h2>
<p>Web browsers receive HTML documents from a web server or from local storage and render the documents into multimedia web pages. HTML describes the structure of a web page semantically and originally included cues for the appearance of the document.
</p>
<p style="margin-top: 40px;">HTML can embed programs written in a scripting language such as JavaScript, which affects the behavior and content of web pages. Inclusion of CSS defines the look and layout of content. The World Wide Web Consortium (W3C), former maintainer of the HTML and current maintainer of the CSS standards, has encouraged the use of CSS over explicit presentational HTML since 1997.</p>

<p><a href="http://www.naver.com" target="blank" title="NAVER">네이버</a></p>

</body>
</html>
```

다른 웹페이지를 만들었습니다. <ol>바디를 보시면 링크가 걸린 3개의 태그가 있습니다. 여기에 전부 색깔을 주고 싶다면 어떻게 할까요? 3개니까 그냥 하면 되겠죠?ㅎㅎ 맞습니다. 적은 개수에는 일일이 해도 되겠지만 만약 대형사이트같은 링크가 수백~수천개씩 걸린 사이트라면? 손가락에 관절염걸리겠죠ㅎㅎ..

```
    <style>
        a{
            color: red;
            text-decoration: none;
        }


    </style>
```

<style> 바디에 보시면 a하고 중괄호가 있네요! 이것은 a태그가 들어간 모든 라인에 영향을 주는 것입니다. 마치 상속과도 같은 개념이죠! 여기서부터가 CSS시작인데 확실히 HTML보다 여러가지 기능이 있을것같습니다.!

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Style/1.png)

이것저것 실험해봤습니다 ㅎㅎ 사진은 그냥 인터넷에 돌아다니는거 받았어요 신세휘님이라고 하네요 ^~^

오늘은 여기까지 하겠습니다!