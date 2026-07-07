---
title: "[Frontend] Javascript로 다크모드 만들기"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 버튼 2개(**야간/주간 모드**)를 만들어, **JavaScript**로 웹페이지의 배경·글자색을 바꿔본다. HTML(구조)·CSS(디자인)에 이어 **JS(동작)**를 더하는 것이다.

---

## 1. 버튼으로 색 바꾸기

```html
<!-- night 버튼 -->
<input type="button" value="night" onclick="
    document.querySelector('body').style.backgroundColor = 'black';
    document.querySelector('body').style.color = 'white';
">

<!-- day 버튼 -->
<input type="button" value="day" onclick="
    document.querySelector('body').style.backgroundColor = 'white';
    document.querySelector('body').style.color = 'black';
">
```

> 💡 **`document.querySelector('body')`**는 인자로 넣은 부분 전체에 영향을 줄 수 있는 실행문이다. `'body'`를 넣으면 **body 전체**에 영향을 준다. `onclick`으로 버튼을 눌렀을 때 실행할 JS를 지정한다.

| 요소 | 역할 |
|------|------|
| `onclick="..."` | 클릭 시 실행할 JS |
| `querySelector('body')` | body 요소 선택 |
| `.style.backgroundColor` | 배경색 변경 |
| `.style.color` | 글자색 변경 |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-Night-Mode/1.png)
![Desktop View](/assets/img/Frontend/HTML-CSS-JS/JS-Night-Mode/2.png)

night 버튼은 배경 검정·글자 흰색, day 버튼은 그 반대로 바뀐다.

---

## 2. 선택자 복습 (class / id / 태그)

```html
<style>
    .js    { font-weight: bold; color: blue; }   /* class */
    .first { color: green; }
    #first { color: yellowgreen; }               /* id */
    span   { color: red; }                       /* 태그 */
</style>
```

| 선택자 | 문법 | 대상 |
|:---:|:---:|------|
| **태그** | `span` | 모든 span |
| **class** | `.js` | class="js"인 것들 (여러 개 가능) |
| **id** | `#first` | id="first"인 것 (고유) |

> 💡 `class`는 **여러 요소에 공유**할 수 있고, `id`는 **한 요소에만 고유**하게 붙인다. CSS 선택자로 스타일 대상을 정한다.

---

## 📝 정리

```
JS 다크모드
├─ 버튼      <input type="button" onclick="JS">
├─ 선택      document.querySelector('body')
├─ 스타일    .style.backgroundColor / .color
└─ 선택자    태그 / .class / #id
```

| 개념 | 한 줄 정의 |
|------|------|
| **onclick** | 클릭 시 JS 실행 |
| **querySelector** | 요소를 선택 |
| **class vs id** | 공유(다수) vs 고유(하나) |

HTML(구조)·CSS(디자인)에 **JS(동작)**를 더하면 페이지가 상호작용한다. `querySelector`로 요소를 잡아 `style`을 바꾸는 것이, 모든 동적 UI의 기본 패턴이다.
