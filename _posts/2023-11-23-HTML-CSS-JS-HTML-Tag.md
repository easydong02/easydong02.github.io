---
title: [Frontend] HTML 태그들
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [html, tag]
render_with_liquid: false
future: true
---

```
<!DOCTYPE html>     <!--HTML5입장-->
    <head>
            <Meta charset="UTF-8">
            <title>프로그래밍</title>
    </head>
        <body>
            <h1>Hello World</h1>
            <br>
            by Denzel Shin<br>
            <a href="http://www.naver.com" target='blank'>네이버</a><br>



            <img src ="image/mine.jpg" width="300px" height="300px">
            <p>Nice to meet <mark>you</mark></p> <!-- p는 문단의 p-->
            <ol>
                <li>순서가 있는 리스크 1</li>
                <li>순서가 있는 리스크 2</li>
                <li>순서가 있는 리스크 3</li>
            </ol>

            <ul>
                <li>순서가 없는 리스크 1</li>
                <li>순서가 없는 리스크 2</li>
                <li>순서가 없는 리스크 3</li>
            </ul>

            <span style = "color:red;">호라락쏘보스</span>
            <!-- 주석은 컨트롤 슬래시 -->
            <input type="text" value ="이름">
            <input type="submit" value ="제출">
            <span>당신의 취미는 무엇인가요??</span>
            <input type="checkbox" >영화감상
            <input type="checkbox" >사진
            <input type="checkbox" >운동

        </body>
    
</html>
```

여러가지를 했습니다 ㅋㅋ 프론트엔드쪽은 결과물을 바로바로 눈으로 확인할 수 있어서 또 다른 재미가 있네용

```
<a href="http://www.naver.com" target='blank'>네이버</a><br>
```

하이퍼링크입니다. 인터넷 주소값을 입력하고 target="blank"로 새창에서 저 링크를 열어줍니다~ 주소는 네이버에요!

```
<img src ="image/mine.jpg" width="300px" height="300px">
```

이미지 링크를 가져와서 웹페이지에 출력하는 건데요. 원래는 주소값이 있어야하지만 제가 사진을 다운받아서 제 디렉토리에 넣었기 때문에 저런식으로 패키지이름으로 경로로 접근합니다\` 너비와 높이를 px단위로 정해주었습니다!

```
<ol>
                <li>순서가 있는 리스트 1</li>
                <li>순서가 있는 리스트 2</li>
                <li>순서가 있는 리스트 3</li>
            </ol>
```

ol은 ordered line? 이겠죠 ?ㅎㅎ; 아무튼 순서가 있는 라인들입니다. 따라서 순번을 1부터 알아서 매겨줍니다

```
            <ul>
                <li>순서가 없는 리스트 1</li>
                <li>순서가 없는 리스트 2</li>
                <li>순서가 없는 리스트 3</li>
            </ul>
```

ul은 unordered line.... 일겁니다!ㅋㅋ.. 순서가 없는거에요

```
<span style = "color:red;">호라락쏘보스</span>
```

이건 호라락쏘보스라는 의식의 흐름으로 만든 글자들에 style, 즉 데코레이션을 주었습니다. 색깔을 빨간색으로 했네요 ㅎㅎ span은 비유하자면 하나의 방? 그런 개념입니다. 문자가 들어갈 수도, 이미지 파일일수도, 아니면 뭐 다른거라던지.. 다 들어갈 수 있습니다.

```
            <input type="text" value ="이름">
            <input type="submit" value ="제출">
            <span>당신의 취미는 무엇인가요??</span>
            <input type="checkbox" >영화감상
            <input type="checkbox" >사진
            <input type="checkbox" >운동
```

input은 html에서 지원하는 여러가지 컴포넌트를 삽입할 수 있습니다! 텍스트 필드와 버튼 그리고 3개의 체크박스를 만들었습니다~

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/HTML-Tag/1.png)

점점 답이 없어지는 내 홈페이지.. ㅎㅎ; 언젠간 이게 네이버가 되겠죠?

이번시간에는 여기까지 하겠습니다!