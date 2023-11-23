---
title: Javascript 원버튼 다크모드
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript]
render_with_liquid: false
future: true
---

저번시간에는 버튼을 2개 만들어서 야간모드와 주간모드를 설정했는데 이번엔 원버튼으로 했습니다!

```
  <input id ="night_day" type = "button" value = "night" onclick = 
  "
  var target = document.querySelector('body')
  if(this.value === 'night') {
    target.style.backgroundColor = 'black';
    target.style.color = 'white';  
   this.value = 'day';
  } else {
    target.style.backgroundColor = 'white';
    target.style.color = 'black';  
   this.value = 'night';
  }   
  ">
```

이 전체가 하나의 버튼입니다. target이라는 document.querySelector('body')를 담는 변수를 만들어 코드가 의미없이 길어지는 것을 피했습니다. 그리고 자바처럼 this를 쓸 수 있습니다. 특이한 점은 여기서 같다라는 비교연산은 '===' 처럼 3개를 씁니다! 그리고 눌렀을 때 this.value를 바꾸어서 원버튼 형식으로 했습니다~

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/1.png)

저기서 버튼을 누르면?

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/2.png)

잘 바뀌네요 ㅎㅎ 재밌습니다~

**<script>**영역에선 프로그래밍 언어처럼 쓸수 있죠? 이것을 통해서 간단한 문자열 배열을 만들어봤습니다.

```
    <script>
        var coworkers = ['egoing','egoing','egoing']
        document.write(coworkers)
        coworkers.push("\nddfdf")
        document.write(coworkers)
    </script>
```

여기는 sql처럼 문자열을 var로 합니다!그리고 출력문은 document.write()로 합니다! 파이썬에서 append()와 같은 역할을 하는 것은 push()입니다. 이러면 여기에 인자로 들어간 것이 마지막에 추가가 되죠!

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/3.png)

잘 출력이 됩니다~

이번 시간에는 여기까지 하겠습니다!