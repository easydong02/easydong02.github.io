---
title: "[Frontend] CSS 그리드(grid)"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, grid]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 2차원 레이아웃 도구인 **CSS Grid**와, 화면 크기에 반응하는 **미디어 쿼리**로 웹페이지를 다듬어본다. **레이아웃**에 초점을 맞춘다.

---

## 1. 선택자 우선순위 — class vs id

```css
.saw    { color: gray; }     /* class → 온점(.) */
#active { color: purple; }   /* id → 샵(#) */
```

```html
<li><a href="1.html" class="saw">HTML</a></li>
<li><a href="2.html" class="saw" id="active">CSS</a></li>  <!-- id가 우선 → purple -->
```

| 선택자 | 문법 | 우선순위 |
|:---:|:---:|:---:|
| class | `.saw` | 낮음 |
| **id** | `#active` | **높음** |

> 💡 한 태그에 같은 옵션이 여럿 걸리면, **더 개별적인 id가 우선**된다. 그래서 위의 CSS 링크는 회색(class)이 아니라 보라색(id)이 된다.

---

## 2. Grid 레이아웃

```css
#grid {
    display: grid;
    grid-template-columns: 150px 1fr;   /* 왼쪽 150px, 오른쪽 나머지 */
}
```

> 💡 `display: grid`로 2차원 격자 레이아웃을 만든다. `grid-template-columns: 150px 1fr`은 **첫 열은 150px 고정, 둘째 열은 남은 공간(1fr) 전부**를 뜻한다. 사이드바(메뉴) + 본문 구조에 적합하다.

```
┌─────────┬──────────────────┐
│  150px  │       1fr         │
│  (메뉴)  │      (본문)        │
└─────────┴──────────────────┘
```

---

## 3. 미디어 쿼리 — 반응형

```css
@media (max-width: 800px) {
    #grid    { display: block; }       /* 격자 → 세로 쌓기 */
    #grid ol { border-right: none; }   /* 테두리 제거 */
    h1       { border-bottom: none; }
}
```

> 💡 **미디어 쿼리**는 반응형 옵션이다. `(max-width: 800px)`는 "화면 폭이 **800px 이하**가 되면" 안쪽 스타일을 적용한다. 여기선 격자(grid)를 세로 쌓기(block)로 바꾸고 테두리를 없앤다.

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Grid/1.png)
![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Grid/2.png)

창 크기를 줄이니 구분선이 사라지고 배치가 `block`으로 바뀐다. (`none`으로 했다면 아예 안 보였을 것)

---

## 📝 정리

```
CSS Grid & 반응형
├─ 선택자    .class(낮음) < #id(높음)
├─ Grid      display:grid, grid-template-columns: 150px 1fr
└─ 미디어쿼리  @media(max-width:800px){ ... } 반응형
```

| 개념 | 한 줄 정의 |
|------|------|
| **id 우선순위** | 개별적일수록 우선 |
| **grid** | 2차원 격자 레이아웃 |
| **1fr** | 남은 공간의 비율 |
| **미디어 쿼리** | 화면 크기별 반응형 스타일 |

Grid는 2차원 레이아웃을, 미디어 쿼리는 반응형을 담당한다. `150px 1fr`처럼 고정+가변을 조합하고, `@media`로 화면 크기에 대응하면 다양한 기기에서 자연스러운 페이지가 완성된다.
