---
title: "[Frontend] Javascript 원버튼 다크모드"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript]
render_with_liquid: false
future: true
---

## 📌 들어가며

저번엔 버튼을 **2개**(야간/주간) 만들었다. 이번엔 **원버튼**으로, 하나의 버튼이 상태를 토글하도록 개선한다.

---

## 1. 원버튼 토글

```html
<input id="night_day" type="button" value="night" onclick="
    var target = document.querySelector('body');
    if (this.value === 'night') {
        target.style.backgroundColor = 'black';
        target.style.color = 'white';
        this.value = 'day';        // 다음 클릭을 위해 값 변경
    } else {
        target.style.backgroundColor = 'white';
        target.style.color = 'black';
        this.value = 'night';
    }
">
```

핵심 아이디어를 정리하면:

| 요소 | 역할 |
|------|------|
| `var target = ...` | 반복되는 `querySelector`를 변수로 (코드 간결) |
| `this` | 자바처럼 자기 자신(버튼) 참조 |
| `===` | JS의 값 비교는 **등호 3개** |
| `this.value = ...` | 버튼 값을 바꿔 다음 클릭 때 반대 동작 |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/1.png)
![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/2.png)

> 💡 버튼을 누를 때마다 `this.value`가 `night`↔`day`로 바뀌면서, 하나의 버튼이 **상태를 토글**한다. `===`는 JS 특유의 엄격한 비교 연산자(값+타입)다.

---

## 2. script 영역 — JS로 배열 다루기

`<script>` 영역에서는 프로그래밍 언어처럼 코드를 쓸 수 있다.

```html
<script>
    var coworkers = ['egoing', 'egoing', 'egoing'];
    document.write(coworkers);      // 출력
    coworkers.push("\nddfdf");      // 배열 마지막에 추가
    document.write(coworkers);
</script>
```

| JS | 역할 | 비교 |
|:---:|------|------|
| `var` | 변수 선언 | (타입 없음) |
| `document.write()` | 출력 | print |
| `push()` | 배열 끝에 추가 | 파이썬의 append() |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-One-Button/3.png)

---

## 📝 정리

```
원버튼 다크모드
├─ 토글    if (this.value === 'night') ... else ...
├─ this    버튼 자기 자신 참조
├─ ===     JS의 엄격한 값 비교
└─ script  var 선언 / document.write / push
```

| 개념 | 한 줄 정의 |
|------|------|
| **this** | 이벤트가 발생한 요소 자신 |
| **===** | 값+타입 모두 비교 |
| **push()** | 배열 끝에 추가 |

원버튼 토글은 "상태를 기억하고 바꾸는" 상호작용의 기본이다. `this`로 요소 자신을 참조하고 `===`로 상태를 판단하는 패턴은, 앞으로 만들 모든 인터랙션의 씨앗이 된다.
