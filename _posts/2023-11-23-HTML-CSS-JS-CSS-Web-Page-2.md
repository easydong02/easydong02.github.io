---
title: "[Frontend] CSS 웹페이지 꾸미기"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [javascript, css]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 저번에 만든 웹페이지에 **JS 슬라이드쇼**와 **CSS 탭 메뉴**를 추가해 그럴듯하게 완성한다. 전체 코드는 길지만, **핵심 부분** 위주로 살펴본다.

> **페이지 구조 한눈에 보기**

```
container
├─ header   (로고 + 드롭다운 메뉴)
├─ slideShow (JS 슬라이드쇼 + prev/next 버튼)
├─ contents (탭 메뉴: 공지/갤러리 + 링크)
└─ footer   (하단 메뉴 + SNS)
```

---

## 1. 슬라이드쇼 (JavaScript)

가장 큰 사진 영역에서 **버튼으로 사진을 넘기는** 슬라이드쇼다.

```html
<div id="slides">
    <img src="images/photo-1.jpg">
    <img src="images/photo-2.jpg">
    <img src="images/photo-3.jpg">
    <button id="prev">&lang;</button>
    <button id="next">&rang;</button>
</div>
```

```javascript
var slides = document.querySelectorAll("#slides > img");
var current = 0;

showSlides(current);
document.getElementById("prev").onclick = prevSlide;
document.getElementById("next").onclick = nextSlide;

function showSlides(n) {
    for (let i = 0; i < slides.length; i++)
        slides[i].style.display = "none";   // 전부 숨기고
    slides[n].style.display = "block";       // n번만 표시
}
function prevSlide() {
    current = (current > 0) ? current - 1 : slides.length - 1;
    showSlides(current);
}
function nextSlide() {
    current = (current < slides.length - 1) ? current + 1 : 0;
    showSlides(current);
}
```

> 💡 핵심은 **`current`(현재 인덱스)로 사진을 관리**하는 것이다. `showSlides(n)`은 모두 `display:none`으로 숨긴 뒤 n번만 `block`으로 보인다. prev/next 버튼이 `current`를 ±1하며 순환한다. (도서관리 앱의 페이지 전환과 같은 원리)

---

## 2. 탭 메뉴 (CSS만으로)

공지사항/갤러리를 JS 없이 **라디오 버튼 + CSS**로 전환한다.

```html
<input type="radio" id="tab1" name="tabs" checked>
<label for="tab1">공지사항</label>
<input type="radio" id="tab2" name="tabs">
<label for="tab2">갤러리</label>

<div id="notice" class="tabContent">...</div>
<div id="gallery" class="tabContent">...</div>
```

```css
#tabMenu input[type="radio"] { display: none; }   /* 라디오 버튼 숨김 */
.tabContent { display: none; }                     /* 내용 숨김 */

/* 체크된 라디오에 따라 해당 내용 표시 */
#tab1:checked ~ #notice,
#tab2:checked ~ #gallery { display: block; }

/* 선택된 탭 제목 강조 */
#tabMenu input:checked + label { background-color: #eee; }
```

> 💡 **`:checked` + 형제 선택자(`~`)**가 마법이다. 라디오 버튼은 숨기고 label만 보이게 한 뒤, "tab1이 체크되면 notice를 보이고, tab2가 체크되면 gallery를 보인다"를 **순수 CSS로** 구현했다. JS 없이 탭이 동작한다.

| 선택자 | 역할 |
|:---:|------|
| `input:checked` | 체크된 라디오 |
| `+ label` | 바로 다음 형제 |
| `~ #notice` | 뒤의 형제 요소 |

---

## 3. 자주 쓴 CSS 기법

| 속성 | 역할 |
|------|------|
| `* { margin:0; padding:0; box-sizing:border-box }` | 기본 스타일 초기화 |
| `float: left/right` | 요소를 좌우로 흐르게 배치 |
| `overflow: hidden` | 영역을 넘치는 부분 숨김 |
| `position: absolute` | 버튼을 이미지 위 절대 위치에 |
| `border-radius: 100%` | 원형(아이콘) |
| `ul { list-style: none }` | 리스트 기호 제거 |
| `:last-child` | 마지막 요소만 (구분선 제거) |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Web-Page-2/1.png)

완성된 페이지는 **공지/갤러리 탭 전환**, **사진 위 버튼으로 슬라이드 전환**이 모두 동작한다.

---

## 📝 정리

```
CSS 웹페이지 완성
├─ 구조     header/slideShow/contents/footer
├─ 슬라이드  JS: current 인덱스 + showSlides(display 토글)
├─ 탭       CSS: input:checked ~ 로 순수 CSS 전환
└─ 레이아웃  float / position / overflow / border-radius
```

| 개념 | 한 줄 정의 |
|------|------|
| **슬라이드쇼** | 인덱스로 display 토글 |
| **`:checked ~`** | 라디오 상태로 콘텐츠 전환 |
| **float/position** | 요소 배치 |

이 페이지는 HTML(구조)·CSS(디자인)·JS(동작)가 모두 어우러진 결과물이다. 특히 **CSS만으로 탭을 구현**하는 `:checked ~` 기법, **JS 인덱스로 슬라이드**를 넘기는 패턴은 실전에서 두고두고 쓰인다. 프론트엔드는 "이런 게 있구나"만 알아도 이해가 쏙쏙 되어 좋았다.
