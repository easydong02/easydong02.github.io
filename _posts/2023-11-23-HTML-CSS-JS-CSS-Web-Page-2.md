---
title: "[Frontend] CSS 웹페이지 꾸미기"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript, css]
render_with_liquid: false
future: true
---

이번 시간에는 저번에 만들었던 웹페이지에 js도 추가하여 그럴듯하게 만들어봤습니다. 일단 지금까지 한 모든 코드를 올리겠습니다. 다 볼 필요는 없고 중요한 부분만 볼게요!

HTML

```
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome Home</title>
    <link rel="stylesheet" href="css/style.css"
</head>
<body>
    <div id="container">
        <header>
            <div id="logo">
                <a href="index.html"><h1>Welcome Home</h1></a>
            </div>
            <nav>
                <ul id="topMenu">
                    <li><a href="#">소개</a></li>
                    <li><a href="#">프론트<span>▼</span></a>
                        <ul>
                            <li><a href="#">HTML</a></li>
                            <li><a href="#">CSS</a></li>
                            <li><a href="#">JS</a></li>
                        </ul>
                    </li>
                    <li><a href="#">자바<span>▼</span></a>
                        <ul>
                            <li><a href="#">JSP</a></li>
                            <li><a href="#">Spring</a></li>
                            <li><a href="#">Spring Boot</a></li>
                        </ul>
                    </li>
                    <li><a href="#">파이썬<span>▼</span></a>
                        <ul>
                            <li><a href="#">Flask</a></li>
                            <li><a href="#">Django</a></li>
                            <li><a href="#">PyQt</a></li>
                        </ul>
                    </li>
                    <li><a href="#">문의</a></li>


                </ul>

            </nav>
        </header>
        <div id = "slideShow">
            <div id="slides">
                <img src="images/photo-1.jpg" alt="">
                <img src="images/photo-2.jpg" alt="">
                <img src="images/photo-3.jpg" alt="">
                <button id="prev">&lang;</button>
                <button id="next">&rang;</button>
            </div>
        </div>
        <div id="contents">
            <div id ="tabMenu">
                <input type="radio" id="tab1" name="tabs" checked>
                <label for="tab1">공지사항</label>
                <input type="radio" id="tab2" name="tabs">
                <label for="tab2">갤러리</label>       
                
                <div id="notice" class="tabContent">
                    <h2>공지사항 내용입니다.</h2>
                    <ul>
                        <li>사무실을 이전했습니다.</li>
                        <li>[참가 모집]체험 실습 초대</li>
                        <li>[참가 모집]여름 방학 체험 모집</li>
                        <li>추천 여행지</li>
                    </ul>
                </div>
                <div id="gallery" class="tabContent">
                    <h2>갤러리 내용입니다.</h2>
                    <ul>
                        <li><img src="images/img-1.jpg" alt=''></li>
                        <li><img src="images/img-2.jpg" alt=''></li>
                        <li><img src="images/img-3.jpg" alt=''></li>
                    </ul>
                </div>
            </div>
            <div id = "links">
                <ul>
                    <li>
                        <a href = "#"><span id = "quick-icon1"></span>
                            <p>페이스북</p></a>
    
                    </li>
                    <li>
                        <a href = '#'><span id = "quick-icon2" ></span>
                            <p>유튜브</p></a>
                    </li>
                    <li>
                        <a href ='#'><span id="quick-icon3" ></span>
                            <p>인스타</p></a>
                    </li>
                </ul>
            </div>
        </div>
        <footer>
            <div id ='bottomMenu'>
                <ul>
                    <li><a href='#'>소개</a></li>
                    <li><a href='#'>개인정보</a></li>
                    <li><a href='#'>약관</a></li>
                    <li><a href='#'>사이트맵</a></li>
                </ul>
                <div id="sns">
                    <ul>
                        <li><a href='#'><img src="images/sns-1.png" alt=''></a></li>
                        <li><a href='#'><img src="images/sns-2.png" alt=''></a></li>
                        <li><a href='#'><img src="images/sns-3.png" alt=''></a></li>
                    </ul>
                </div>
            </div>

        </footer>
    </div>
    <script src="js/slideshow.js"></script>
</body>
</html>
```

style.css

```
@import url('https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap');






* {
    margin: 0;  
    /* 마진 초기화 */
    padding: 0; 
    /* 패딩 초기화 */
    box-sizing:border-box; 
    /* 박스 영역을 테두까지 지정 */
}

#container {
    margin: 0 auto;
    /* 화면 중앙에 배치 */
    width: 1200px;
    /* 너비 지정 */
}

header {
    width:100%;
    height:100px;
    background-color: blue;

}
#logo {
    float:left;
    width:250px;
    height:100px;
    line-height: 100px;
    padding-left:30px;

}

#logo h1 {
    font-family: 'Indie Flower', cursive;
    color:white;
    font-size:35px;
}


nav {
    float:right;
    width:900px;
    height:100px;
    padding-top:40px;
    /* 메뉴를 약간 아래로 위치 */
}

#topMenu {
    height: 60px;
}

#topMenu > li {
    float:left;
    position:relative;

}
#topMenu > li > ul {
    display: none;
    /* 일단 감추자 */
    position: absolute;
    width:160px;
    background-color:whitesmoke;
    left:0;
    margin:0;
}


#topMenu > li > ul > li {
    padding:10px 10px 10px 30px;
}


#topMenu > li > ul > li > a{
    font-size:14px;
    padding:10px;
    color:black;
}

#topMenu > li:hover > ul {
    display: block;
}

#topMenu > li > ul > li > a:hover {
    color:brown;
}

#topMenu > li > a {
    display: block;
    color:white;
    text-shadow: 1px yellow;
    padding:20px;

}
#topMenu > li > a > span {
    font-size: 0.5em;

}


#topMenu > li > a:hover {
    color:silver;

}

#slideShow {
    clear: both;
    width:100%;
    height:300px;
    overflow:hidden;
    /* 영역에서 넘치면 감추자 */
    position:relative;
}

#slides {
position: absolute;
}

#slides > img {
    width:100%;
    float:left;
}

button {
    position:absolute;
    /* width값을 설정하지 않을 경우 100%가 아닌 내용있는 위치까지 */
    height: 100%;
    top: 0;
    border:none;
    /* 테두리는 없이 */
    padding: 20px;
    background-color: transparent;
    color:black;
    font-weight: 800;
    font-size: 24px;
    opacity: 0.5;
}

#prev {
    left: 0;
}


#next {
    right: 0;
}

button:hover {
    background-color:darkgray;
    color:white;
    opacity: 0.5;
    cursor:pointer;
}

#contents {
    width:100%;
    height:300px;
}


#tabMenu {
    float:left;
    width:600px;
    height:100%;
}

#tabMenu input[type="radio"] {
    display: none;
    /* 버튼 감추기 */
}

#tabMenu label {
    display: inline-block;
    /* 가로로 배치 */
    margin: 0 0;
    padding: 15px 25px;
    font-weight: 600;
    text-align:center;
    color:tomato;
    border: 1px;
}

#tabMenu label:hover {
    color:thistle;
    cursor:pointer;
}
/* 선택된 탭의 제목 스타일 */
#tabMenu input:checked  + label {
    color: grey;
    border: 1px solid #ddd;
    background-color: #eee;
}

.tabContent {
    display:none;
    padding: 20px 0 0;

}

    
#tab1:checked ~ #notice,
#tab2:checked ~ #gallery {
    display:block;
}


.tabContent h2{
    display:none;
}

#notice ul{
    margin-left: 30px;
}

#notice ul li{
    font-size: 16px;
    line-height:2.5;
}

#gallery ul li{
    display: inline;
}
#links {
    float:right;
    width:600px;
    height:100%;
    
}

#links ul{
    overflow:hidden;
}

#links ul li{
    float: left;
    width:30%;
    text-align:center;
    margin: 10px;
}

#links ul li a span{
    display: block;
    margin:0 auto;
    /* 가운데 배치 */
    width: 100px;
    height:100px;
    border-radius:100%;
    border:1px solid #ddd;
    line-height: 150px;

}

#quick-icon1{
    background-image: url(../images/sns-1.png);
}
#quick-icon2{
    background-image: url(../images/sns-2.png);
}
#quick-icon3{
    background-image: url(../images/sns-3.png);
}

#links ul li a p{
    margin-top:15px;
    font-weight:600;
    color:indianred;
}

footer{
    width:100%;
    height: 100px;
    border-top:2px solid black;
}

#bottomMenu ul li{
    float:left;
    border-right: 1px solid #222;
}

#bottomMenu ul li:last-child{
    border-right: none;
}



a {
    text-decoration:none;
}

ul {
    list-style: none; 
    /* 기호생략 */
}


```

js

```
var slides = document.querySelectorAll("#slides > img");
var prev = document.getElementById("prev");
var next = document.getElementById("next");
var current = 0;

showSlides(current);
prev.onclick = prevSlide;
next.onclick = nextSlide;

function showSlides(n) {
  for (let i = 0; i < slides.length; i++) {
    slides[i].style.display = "none";
  }
  slides[n].style.display = "block";
}

function prevSlide() {
  if (current > 0) current -= 1;
  else
    current = slides.length - 1;
    showSlides(current);
}

function nextSlide() {
  if (current < slides.length - 1) current += 1;
  else
    current = 0;
    showSlides(current);  
}
```

여기서 볼것은 js 구문입니다. 화면에 슬라이드쇼를 해서 버튼으로 원하는 사진으로 넘기고 싶은데요 버튼을 2개만듭니다. prev 와 next인데요. showSlides()라는 함수를 만들고 인자에는 원하는 사진만 띄우고 나머진 display: none;으로 했습니다. 저번에 도서관리프로그램 만들때와 같습니다 ㅎㅎ 그리고 prevSlide()와 nextSlide()를 만들어 각각의 버튼과 연동을 했습니다. current를 사진의 인덱스 번호로 해서 그것을 갖고 노는 함수입니다.

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Web-Page-2/1.png)

오늘까지한 웹페이지 입니다. 공지사항과 갤러리를 바꿔서 볼수 있으며 가장 큰 사진에 커서를 대면 버튼이 양끝에 나타나고 그것을 눌렀을 때 다른 사진으로 바뀝니다!

사실 프론트엔드쪽은 깊이 배우기보단 이게 있구나 정도로 해도 이해가 쏙쏙 되서 좋았습니다!