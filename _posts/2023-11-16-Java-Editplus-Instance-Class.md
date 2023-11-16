---
title: Editplus 객체 클래스
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

저번 시간 부터 이클립스는 잠시 안쓰기로 했었죠? 그래서 국산 에디터인 editplus를 사용하겠습니다!

![Desktop View](/assets/img/Programming-Language/Java/Editplus-Instance-Class/1.png)

에딧 플러스는 이렇게 생겼습니다 ㅎㅎ 약간 원시적인 느낌이 듭니다. 메모장 상위호환느낌?

자그럼 이제 오늘 배웠던 거를 써보겠습니다. 사실 JAVA에서 객체, 클래스 등 객체지향개념이 엄청 중요하죠. 따라서 그 개념을 제대로 이해하기 위해서 이번에는 java를 사용하지 않고 html방식으로 js를 사용하여 객체개념을 이해해볼게요. 한마디로 저만의 홈페이지를 만들어 거기서 노는겁니다~

html소스코드를 쭈욱 올려보겠습니다~

```
<!doctype html>
 <head>
  <meta charset="UTF-8">
  <title>클래스 개념</title>
  
 </head>
 <body>
 <button onClick="moveAll()">화살 날리기</button>
 </body>
</html>

<script>
//2015 년도에 발표된 ECMAScript(js명칭)자바와 흡사한 class를 지원
//화살 자체가 아닌, 장차 화살을 만들어 낼 수 있는 틀인 클래스를 정의해보자!!
//OOP언어 = 객체지향 언어 = 객체가 중심이 되는 언어..
class Arrow{
	constructor(x,y,vel,color){//ES에서는 생성자의 명칭이 정해져 있다...
this.s=document.createElement("span");//프로그래밍적으로 생성
this.s.innerText="→";//<span>→</spn>
this.x=x;
this.y=y;
this.vel=vel;//속도
this.color= color;
//원하는 좌표로 이동
this.s.style.position="absolute";//절대위치로 지정
this.s.style.left=this.x+"px";
this.s.style.top=this.y+"px";
this.s.style.fontSize=30+"px";
this.s.style.color = this.color;

//생성된 span을 body에 자식요소로 붙이자
document.body.appendChild(this.s);
}
	move(){
	
	//문서 내의 s라 불리는 녀석의 스타일 위치값중 left를 점점 증가 시키겠다.
	this.x +=this.vel;
	this.s.style.left=this.x+"px";
	}
}
//문서 내의 span이라는 태그를 날려보자!
var a,b;

function moveAll(){
	a.move();
	b.move();
}
//문서가 로드되면, 그때 init()하겠다..
window.addEventListener("load",function(){
	a = new Arrow(100,200,10,"red");
	b = new Arrow(250,300,15,"blue");
});
</script>
```

자바에서 보는 문법이랑 많이 다르죠? 여기는 html이라 그런 것같습니다. 보통 태그로 이루어졌는데 html에 대해서 잘 모르는 제가 어떻게 사용할 수있었을까요? 바로
```
<script> </script>
```
파트 부분에서는 우리가 배우는 객체지향언어인 js를 사용할 수 있기 때문입니다. 그리고 이 js는 우리가 사용하는 자바랑 유사한 점이 많아서 객체와 클래스를 배우기가 좋습니다.

제가 전에도 말씀드렸지만, 클래스와 객체의 관계는 무엇일까요? 바로 붕어빵 틀과 붕어빵의 관계라고 생각하시면 됩니다. 틀이 있으면 재료를 넣고 얼마든지 틀의 형태로 나온 붕어빵을 만들 수 있죠. 따라서 화살을 만들 수 있는 틀을 만들고 화살을 만들어 버튼을 누르면 조금씩 움직이는 홈페이지를 만들어보았습니다~

```
class Arrow{
	constructor(x,y,vel,color){//ES에서는 생성자의 명칭이 정해져 있다...
this.s=document.createElement("span");//프로그래밍적으로 생성
this.s.innerText="→";//<span>→</spn>
this.x=x;
this.y=y;
this.vel=vel;//속도
this.color= color;
//원하는 좌표로 이동
this.s.style.position="absolute";//절대위치로 지정
this.s.style.left=this.x+"px";
this.s.style.top=this.y+"px";
this.s.style.fontSize=30+"px";
this.s.style.color = this.color;

	//생성된 span을 body에 자식요소로 붙이자
	document.body.appendChild(this.s);
}
```

Arrow 클래스를 떼왔습니다. 자바에서는 생성자를 클래스 이름과 같은 것으로 만들었는데 여기선 constructor로 만듭니다. 그리고 매개 변수를 x,y,vel,color로 했습니다 x랑 y는 웹페이지 좌측 최상단을 기점으로 하는 좌표입니다. vel은 화살이 한번 누를 때마다 움직이는 거리를 화살이 날아가는 속도에 비유하여 vel로 했습니다. 그리고 화살표의 색을 color로 했습니다.

```
this.s=document.createElement("span");//프로그래밍적으로 생성
this.s.innerText="→";//<span>→</spn>
```

이것은 생소할텐데 html에서는 <span> </span>이 하나의 칸이라고 할 수 있습니다. 그리고 여기에 우리가 쓸 화살 →을 만들었습니다. 즉,

<span>→</span>를 프로그램적으로 만든 것이죠. 따라서 생성자로 new 연산자를 통해서 객체를 만들 때마다 화살이 생기는 거죠.

```
this.s.style.position="absolute";
this.s.style.left=this.x+"px";
this.s.style.top=this.y+"px";
this.s.style.fontSize=30+"px";
this.s.style.color = this.color;

document.body.appendChild(this.s);
```

이것은 매개변수로 들어온 x,y 좌표로 화살의 위치를 정하고 color로 색을 정하는 것입니다. appendchild는 화살을 만들 때 body에 자식요소로 넣는 작업입니다.

이제 화살을 만드는 틀을 대충 만들었습니다. 이제 화살만 있으면 안 되겠죠? 이 화살을 날리는 메서드를 Arrow클래스에 만들어보겠습니다.

```
move(){
	this.x +=this.vel;
	this.s.style.left=this.x+"px";
	}
```

되게 간단하죠? vel로 들어온 값은 기존의 x값에 누적시킨뒤 그 좌표만큼 왼쪽에서 떨어뜨리는 겁니다. 따라서 반복될수록 x의 값이 누적되어 커지니까 점점 옆으로 가는 것처럼 보이겠습니다.

```
var a,b;

function moveAll(){
	a.move();
	b.move();
}
```

이것은 전역변수 a,b (js는 자료형이 따로 없습니다. 그냥 변수면 다 var를 붙이는 것같더라고요..) 를 선언했습니다. 일단 2개 객체(화살)를 만들기 위해서죠. 그리고 moveAll이란 메소드를 또 만들어서 각각 객체의 move()메소드를 실행시키려고 합니다.

```
//문서가 로드되면, 그때 init()하겠다..
window.addEventListener("load",function(){
	a = new Arrow(100,200,10,"red");
	b = new Arrow(250,300,15,"blue");

});
```

이것은 홈페이지에 들어갔을 대 입니다. 약간 main메서드같은 느낌이죠 ㅎㅎ 그리고 아까 만든 a,b에 new 연산자로 객체 주소값을 넣어줬습니다. 그리고 각각 x,y좌표, 속도, 색깔을 정했습니다.

![Desktop View](/assets/img/Programming-Language/Java/Editplus-Instance-Class/2.png)

화살 두개가 보이네요 ㅎㅎ 그리고 저 버튼은 무엇일까요?

```
<button onClick="moveAll()">화살 날리기</button>
```

바디에 버튼을 눌러서 클릭으로 아까만든 moveAll()메서드를 실행시키는 겁니다. 이제 누를 때마다 화살이 오른쪽으로 갑니다 ^~^

이번 JAVA시간엔 HTMl을 이용해서 객체와 클래스개념을 잘 이해할 수 있었습니다!!