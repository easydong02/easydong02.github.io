---
title: "[Java] Editplus 객체 클래스"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 국산 에디터 **EditPlus**를 소개하고, **객체지향의 핵심 개념(클래스·객체)**을 자바가 아닌 **HTML + JavaScript**로 이해해본다. JS는 자바와 문법이 비슷해 객체·클래스를 배우기 좋다. 화살(→)을 만들어 버튼을 누르면 움직이는 미니 홈페이지를 만들어보자.

![Desktop View](/assets/img/Programming-Language/Java/Editplus-Instance-Class/1.png)

> 💡 EditPlus는 "메모장 상위호환" 느낌의 원시적인 에디터다.

---

## 1. 클래스와 객체 — 붕어빵 틀과 붕어빵

> **클래스와 객체의 관계**는 **붕어빵 틀과 붕어빵**이다. 틀(클래스)이 있으면 재료를 넣어 얼마든지 붕어빵(객체)을 찍어낼 수 있다.

화살 자체가 아니라, **화살을 찍어낼 틀(Arrow 클래스)**을 정의하는 것이 핵심이다.

```
Arrow (클래스/틀)  ──new──►  화살 a (빨강)
                   ──new──►  화살 b (파랑)
```

---

## 2. 전체 코드

```html
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
class Arrow {
    constructor(x, y, vel, color) {   // ES는 생성자 이름이 constructor로 정해짐
        this.s = document.createElement("span");  // <span>을 프로그램적으로 생성
        this.s.innerText = "→";
        this.x = x;
        this.y = y;
        this.vel = vel;               // 속도
        this.color = color;
        // 좌표·스타일 지정
        this.s.style.position = "absolute";
        this.s.style.left = this.x + "px";
        this.s.style.top = this.y + "px";
        this.s.style.fontSize = 30 + "px";
        this.s.style.color = this.color;
        document.body.appendChild(this.s);   // body에 자식으로 추가
    }
    move() {
        this.x += this.vel;                  // x를 vel만큼 누적
        this.s.style.left = this.x + "px";   // 그만큼 오른쪽으로 이동
    }
}

var a, b;
function moveAll() {
    a.move();
    b.move();
}
window.addEventListener("load", function () {   // 문서 로드 시 (main 같은 역할)
    a = new Arrow(100, 200, 10, "red");
    b = new Arrow(250, 300, 15, "blue");
});
</script>
```

---

## 3. 코드 뜯어보기

### 생성자 (constructor)

자바는 생성자 이름을 클래스명과 같게 하지만, **JS(ES)는 `constructor`로 정해져 있다.**

| 매개변수 | 의미 |
|:---:|------|
| `x`, `y` | 웹페이지 좌측 최상단 기준 좌표 |
| `vel` | 한 번 누를 때 이동 거리(속도) |
| `color` | 화살표 색 |

```javascript
this.s = document.createElement("span");
this.s.innerText = "→";
```

> 💡 HTML에서 `<span>`은 하나의 칸이다. 위 코드는 `<span>→</span>`을 **프로그램적으로 생성**한 것이다. 따라서 `new`로 객체를 만들 때마다 화살이 하나씩 생긴다.

### move() 메소드

```javascript
move() {
    this.x += this.vel;                // x 누적
    this.s.style.left = this.x + "px"; // 왼쪽에서 그만큼 떨어뜨림
}
```

`vel`만큼 `x`를 누적해 화살을 옆으로 이동시킨다. 반복될수록 점점 오른쪽으로 가는 것처럼 보인다.

### 실행 흐름

```javascript
window.addEventListener("load", function () {  // 페이지 로드 = main
    a = new Arrow(100, 200, 10, "red");
    b = new Arrow(250, 300, 15, "blue");
});
```

![Desktop View](/assets/img/Programming-Language/Java/Editplus-Instance-Class/2.png)

버튼을 누르면 `onClick="moveAll()"`로 두 화살의 `move()`가 실행되어 옆으로 날아간다.

```html
<button onClick="moveAll()">화살 날리기</button>
```

---

## 📝 정리

```
객체지향 (JS로 이해)
├─ 클래스   Arrow = 화살을 찍어내는 틀 (붕어빵 틀)
├─ 객체     new Arrow(...) = 실제 화살 (붕어빵)
├─ 생성자   JS는 constructor, this로 필드 초기화
└─ 메소드   move() = 객체의 동작
```

| 개념 | 한 줄 정의 |
|------|------|
| **클래스** | 객체를 찍어내는 틀 |
| **객체** | 클래스로 만든 실체 (`new`) |
| **생성자** | 객체 생성 시 필드 초기화 |

자바 대신 JS로 실습하니, **"틀(클래스)로 실체(객체)를 찍어낸다"**는 객체지향의 핵심이 눈에 보이는 화살로 체감됐다. 언어는 달라도 객체지향의 개념은 동일하다.
