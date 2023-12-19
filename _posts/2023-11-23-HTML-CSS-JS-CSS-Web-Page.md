---
title: "[Frontend] CSS로 웹페이지 만들기"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, javascript]
render_with_liquid: false
future: true
---

이번 시간에는 css를 적극 활용하여 메뉴바를 만들었습니다. 코드가 길어서 각각 id부분씩 html과 css로 나눠서 올릴게요!

```
    <div id ='container'>
        <header>
            <div id="logo">
                <a href='index.html'><h1>welcome home</h1></a>
            </div>
            <nav>
                <ul id='topMenu'>
                    <li><a href="#">소개</a></li>
                    <li><a href="#">프론트<span>▼</span></a>
                        <ul>
                            <li><a href="#">HTML</a></li>
                            <li><a href="#">CSS</a></li>
                            <li><a href="#">Java Script</a></li>
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
        <div id='slideShow'>
        </div>
        <div id='contents'>
            <div id ='tabMenu'>
            </div>
            <div id="links">
            </div>   
        </div>
        <footer>
        </footer>  
    </div>
```

```
*{
    margin:0;
    /* 마진 초기화 */
    padding:0;
    box-sizing: border-box;
    /*박스 영역을 테두리까지 지정*/
}

#container{
    margin:0 auto;
    /* 화면 중앙에 배치 */
    width: 1200px;
    /* 너비 지정 */
}
```

위 소스코드는 html이구 밑에는 css입니다. 컨테이너는 모두를 포함하는 div로 했습니다! 대략적인 너비와 간격을 설정했습니다.

```
            <div id="logo">
                <a href='index.html'><h1>welcome home</h1></a>

            </div>
```

```
@import url('https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap');

#logo{
    float:left;
    width:250px;
    height:100px;
    line-height: 100px;
    background-color: brown;
    padding-left:25px;
}

#logo h1{
    font-family: 'Indie Flower', cursive;
    color: white;
    font-size: 35px;
}
```

로고 부분입니다. css로 너비와 높이를 정하고 색과 간격을 정했습니다. 그리고 logo를 id로 쓰는부분 중에 h1과 그 산하부분들도 설정을 해줄 수 있습니다. 폰트를 가져왔는데요. 구글 폰트에서 가져왔습니다 ㅎㅎ

```
                <ul id='topMenu'>
                    <li><a href="#">소개</a></li>
                    <li><a href="#">프론트<span>▼</span></a>
                        <ul>
                            <li><a href="#">HTML</a></li>
                            <li><a href="#">CSS</a></li>
                            <li><a href="#">Java Script</a></li>
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
```

```
#topMenu{
    height:60px;
}

#topMenu > li{
    float:left;
    position:relative;
}

#topMenu > li > ul{
    display: none; 
    /* 일단 감추자 */
    position: absolute;
    width: 160px;
    background-color: whitesmoke;
    left:0;
    margin:0;
}

#topMenu > li > ul > li{
    padding:10px 10px 10px 30px;

}

#topMenu > li > ul > li > a{
    font-size:14px;
    padding:10px;
    color:black;

}

#topMenu > li:hover > ul{
    display: block;
}

#topMenu > li > ul > li > a:hover{
    font-size:14px;
    padding:10px;
    color:rgba(187, 30, 226, 0.952);
    font: bold;
}


#topMenu > li > a{
    display:block;
    color:white;
    text-shadow: 1px yellow;
    padding:20px;
}

#topMenu > li > a > span{
    font-size: 0.5em;
}

#topMenu > li > a:hover{
    color:silver;
}
```

이번 시간 중 가장 중요한 부분입니다. 저는 탑 메뉴바를 만들어서 각각의 메뉴와 그 산하 메뉴를 만들고 평소에는 최상위 메뉴만 보여주고 커서를 갖다댔을 때만 그 밑의 메뉴들을 보여주려고 합니다. 먼저 html로 골격을 만들었습니다. 그리고 css가 좀 긴데요 평소에는 li안에 있는 ul들을 display:none;으로 안보이게했습니다. 여기서 아까 #logo h1과 #logo > h1이랑 뭐가 다를까요? 정답은 >없이 쓰면 그 h1은 물론 h1안에 포함되어 있는 것들도 전부 적용이 되는데 #logo >h1을 하면 딱 logo라는 아이디 안에 있는 h1만 적용이 됩니다. 아무튼. 또 눈여겨 봐야 할것은 li : hover인데요. hover는 둥둥 떠있다라는 뜻을 담고 있잖아요? 마찬가지로 이걸쓰면 마우스를 클릭하지 않고 그 영역에 커서만 대도 실행합니다. 따라서 최상위 메뉴들에게 커서를 대면 display: block;으로 보이게 처리했습니다. 그리고 또 마찬가지로 하위메뉴에 커서를 댔을때도 따로 :hover로 설정하여 글씨색과 font:bold;로 하였습니다. 한번 볼까요?

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Web-Page/1.png)

자 이렇게 웹페이지가 나오고 프론트에 커서를 갖다대면 밑에 하위 목록들이 나오는 것과, 하위 목록에 커서를 대면 색이 바뀌는 것도 확인할 수 가 있습니다! 확실히 프론트엔드부분은 눈에 보이는게 커서 이것저것 해보는 즐거움이 있네요 ㅎㅎ

이번 시간에는 여기까지 하겠습니다!
