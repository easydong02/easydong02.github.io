---
title: "[Frontend] CSS로 웹페이지 만들기"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, javascript]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 CSS를 적극 활용해 **드롭다운 메뉴바**를 만든다. 평소엔 최상위 메뉴만 보이다가, 마우스를 올리면 하위 메뉴가 펼쳐지는 구조다.

---

## 1. 전체 골격 (HTML)

```html
<div id="container">
    <header>
        <div id="logo"><a href="index.html"><h1>welcome home</h1></a></div>
        <nav>
            <ul id="topMenu">
                <li><a href="#">소개</a></li>
                <li><a href="#">프론트<span>▼</span></a>
                    <ul>   <!-- 하위 메뉴 -->
                        <li><a href="#">HTML</a></li>
                        <li><a href="#">CSS</a></li>
                    </ul>
                </li>
                <!-- 자바, 파이썬 메뉴도 동일 구조 -->
            </ul>
        </nav>
    </header>
    <div id="slideShow"></div>
    <div id="contents"></div>
    <footer></footer>
</div>
```

---

## 2. 기본 세팅 (CSS)

```css
* {
    margin: 0;               /* 마진 초기화 */
    padding: 0;
    box-sizing: border-box;  /* 테두리까지 크기에 포함 */
}
#container {
    margin: 0 auto;          /* 화면 중앙 배치 */
    width: 1200px;
}
```

| 속성 | 역할 |
|------|------|
| `* { margin:0; padding:0 }` | 브라우저 기본 여백 초기화 |
| `box-sizing: border-box` | 테두리 포함해 크기 계산 |
| `margin: 0 auto` | 좌우 자동 여백 → 중앙 정렬 |

**구글 폰트**도 가져올 수 있다.

```css
@import url('https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap');
#logo h1 { font-family: 'Indie Flower', cursive; color: white; }
```

---

## 3. 드롭다운 메뉴 — 핵심

```css
#topMenu > li { float: left; position: relative; }

#topMenu > li > ul {
    display: none;           /* 평소엔 하위 메뉴 감춤 */
    position: absolute;
    width: 160px;
}

#topMenu > li:hover > ul {
    display: block;          /* 마우스 올리면 펼침! */
}
```

핵심 개념 두 가지가 있다.

### ① `>` 자식 선택자

| 표기 | 대상 |
|:---:|------|
| `#logo h1` | logo 안의 **모든** h1 (자손 포함) |
| `#logo > h1` | logo의 **직계 자식** h1만 |

> 💡 `>` 없이 쓰면 하위에 포함된 것까지 전부 적용되고, `>`를 쓰면 **바로 아래 자식**에만 적용된다.

### ② `:hover` — 마우스 오버

> `:hover`는 **클릭이 아니라 커서를 올리기만** 해도 실행된다. 그래서 최상위 메뉴에 커서를 대면 `display: none`이던 하위 메뉴가 `display: block`으로 **펼쳐진다.**

```css
/* 하위 메뉴에 커서 대면 색 변경 */
#topMenu > li > ul > li > a:hover {
    color: rgba(187, 30, 226, 0.952);
    font: bold;
}
```

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Web-Page/1.png)

메뉴에 커서를 대면 하위 목록이 나오고, 하위 항목에 커서를 대면 색이 바뀐다.

---

## 📝 정리

```
CSS 드롭다운 메뉴
├─ 초기화   * { margin:0; box-sizing:border-box }
├─ 중앙배치  margin: 0 auto + width
├─ 자식선택  A > B (직계 자식만)
└─ 드롭다운  ul display:none → li:hover > ul display:block
```

| 개념 | 한 줄 정의 |
|------|------|
| **box-sizing: border-box** | 테두리 포함 크기 계산 |
| **`>` (자식 선택자)** | 직계 자식에만 적용 |
| **`:hover`** | 마우스를 올렸을 때 |

드롭다운 메뉴의 핵심은 **`display:none` → `:hover` 시 `display:block`**이라는 단순한 토글이다. 자식 선택자(`>`)와 hover만 이해하면, JS 없이도 인터랙티브한 메뉴를 만들 수 있다.
