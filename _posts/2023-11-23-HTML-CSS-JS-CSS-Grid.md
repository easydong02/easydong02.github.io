---
title: "[Frontend] CSS 그리드(grid)"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, grid]
render_with_liquid: false
future: true
---


이번엔 2차원 구동방향인 grid를 통해서 살짝 웹페이지를 수정해보겠습니다! 오늘은 약간 레이아웃에 초점이 더 맞춰져있습니다!

```
<!DOCTYPE HTML>
<html>



<head>
    <meta charset="utf-8">
    <title>W3C</title>
    <style>
       a{
            color:black;
            text-decoration: none;
            padding : '30px';
            margin : 0;
        }
        h1{
            font-size: 45px;
            text-align: center;
            border-bottom: 1px solid gray;
            margin:0px;
            padding: 20px;
        }
        ol{
            border-right: 1px solid gray;
            width: 100px;
            margin:0;
            padding: 20px;
        }
        
        body{
            margin:0
        }
        @media(max-width: 800px){
                #grid{
                    display:block;
                }
                #grid ol{
                    border-right: none;
                }
                h1{
                    border-bottom: none;
                }

            }
        .saw{
            color:gray;
        }
        #active{
            color:purple
        }
        #grid{
            display: grid;
            grid-template-columns: 150px 1fr;
        }

    </style>
</head>


<body>
    <img src="https://img1.daumcdn.net/thumb/R720x0/?fname=http%3A%2F%2Ft1.daumcdn.net%2Fliveboard%2Ffaibet%2Fe0f37522d0694c1b8cb2b7a1953b6929.jpg" width="20%">
    <h1>WEB</font></h1>
    <div id="grid">
        <ol>
    
            <li><a href="1.html" class=saw>HTML</a></li><br>
            <li><a href="2.html" class=saw id= active>CSS</a></li><br>
            <li><a href="3.html">JS</a></li><br>
        </ol>
    
    <div>
        <h2>CSS란 무엇인가?</h2>
        <p>Web browsers receive HTML documents from a web server or from local storage and render the documents into multimedia web pages. HTML describes the structure of a web page semantically and originally included cues for the appearance of the document.
        </p>
        <p style="margin-top: 40px;">HTML can embed programs written in a scripting language such as JavaScript, which affects the behavior and content of web pages. Inclusion of CSS defines the look and layout of content. The World Wide Web Consortium (W3C), former maintainer of the HTML and current maintainer of the CSS standards, has encouraged the use of CSS over explicit presentational HTML since 1997.</p>
    </div>
    </div>
    </body>

<p><a href="http://www.naver.com" target="blank" title="NAVER">네이버</a></p>

</body>
</html>
```

style태그에 뭐가 많죠? html에는 클래스와 아이디가 있습니다. 아이디가 더 개별적인 느낌이므로 한 태그에 여러 같은 옵션을 부여했을때 아이디가 가장 우선순위가 되는것이죠!

```
        .saw{
            color:gray;
        }
        #active{
            color:purple
        }
```

클래스는 그 클래스를 지정하는 태그에 class =saw로 정하면 style 태그 부분에서 온점(.)을 사용하여 특정 클래스의 옵션을 부여할 수 있습니다. 아이디는 id=active처럼 원하는 아이디를 저런 형식으로 지정할 수 있습니다. 마찬가지로 style태그 부분에서 샵(#)을 사용하여 특정 아이디의 옵션을 부여할 수 있습니다!

```
        @media(max-width: 800px){
                #grid{
                    display:block;
                }
                #grid ol{
                    border-right: none;
                }
                h1{
                    border-bottom: none;
                }

            }
```

이것은 미디어쿼리라는 것인데요 반응형 옵션입니다. (max-width: 800px)는 조건부입니다. max는 ~까지입니다. 따라서 800px크기가 되면 그 안에 있는 것들을 실행시킵니다. 여기선 display를 block으로 하고 border line, 즉 테두리를 없애는데요

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Grid/1.png)

이게 전체화면이라면 여기서 창크기를 줄여봅시다!

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Grid/2.png)

창 크기를 이렇게 줄이니까 줄도 없어지고 디스플레이 방식도 block으로 바뀌었습니다~ 만약 none으로 했으면 아무것도 안 보였겠죠?

이번에는 여기까지 하겠습니다!