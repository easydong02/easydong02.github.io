---
title: "[Frontend] HTML 태그들"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [html, tag]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **HTML의 기본 태그들**을 다룬다. 프론트엔드는 결과물을 바로 눈으로 확인할 수 있어 또 다른 재미가 있다.

---

## 1. 기본 구조

```html
<!DOCTYPE html>          <!-- HTML5 선언 -->
<head>
    <meta charset="UTF-8">
    <title>프로그래밍</title>
</head>
<body>
    <h1>Hello World</h1>
    <br>                  <!-- 줄바꿈 -->
    by Denzel Shin
</body>
</html>
```

| 태그 | 역할 |
|------|------|
| `<!DOCTYPE html>` | HTML5 문서 선언 |
| `<head>` | 메타 정보(제목·인코딩) |
| `<body>` | 실제 화면에 보이는 내용 |
| `<h1>` / `<br>` | 제목 / 줄바꿈 |

---

## 2. 주요 태그

**하이퍼링크** — `target="blank"`로 새 창에서 열기:

```html
<a href="http://www.naver.com" target="blank">네이버</a>
```

**이미지** — `src`에 경로, 크기는 px로:

```html
<img src="image/mine.jpg" width="300px" height="300px">
```

**리스트** — 순서 있음/없음:

```html
<ol>                          <!-- ordered: 1,2,3 자동 번호 -->
    <li>순서가 있는 리스트</li>
</ol>
<ul>                          <!-- unordered: 순서 없음 -->
    <li>순서가 없는 리스트</li>
</ul>
```

| 태그 | 의미 |
|:---:|------|
| `<a href>` | 하이퍼링크 |
| `<img src>` | 이미지 |
| `<ol>` / `<ul>` | 번호 리스트 / 불릿 리스트 |
| `<li>` | 리스트 항목 |

---

## 3. span과 스타일

```html
<span style="color:red;">호라락쏘보스</span>
```

> 💡 **`<span>`**은 하나의 '방' 같은 개념이다. 문자·이미지 등 무엇이든 담을 수 있고, `style`로 데코레이션(색·꾸밈)을 줄 수 있다.

---

## 4. input — 입력 컴포넌트

```html
<input type="text" value="이름">        <!-- 텍스트 필드 -->
<input type="submit" value="제출">       <!-- 버튼 -->
<input type="checkbox">영화감상          <!-- 체크박스 -->
<input type="checkbox">사진
```

| type | 컴포넌트 |
|:---:|------|
| `text` | 텍스트 입력 |
| `submit` | 제출 버튼 |
| `checkbox` | 체크박스 |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/HTML-Tag/1.png)

---

## 📝 정리

```
HTML 태그
├─ 구조    DOCTYPE / head / body
├─ 링크    <a href target="blank">
├─ 이미지  <img src width height>
├─ 리스트  <ol>(번호) / <ul>(불릿) + <li>
├─ span    무엇이든 담는 인라인 방 (+ style)
└─ input   text / submit / checkbox
```

| 태그 | 한 줄 정의 |
|------|------|
| **`<a>`** | 하이퍼링크 |
| **`<ol>/<ul>`** | 순서 O / X 리스트 |
| **`<span>`** | 인라인 요소 담는 방 |
| **`<input>`** | 입력 컴포넌트 |

HTML은 웹페이지의 **구조와 내용**을 담당한다. 태그 하나하나가 화면에 즉시 반영되니, 직접 써보며 익히는 게 가장 빠르다. 언젠가 이게 네이버가 되겠지!
