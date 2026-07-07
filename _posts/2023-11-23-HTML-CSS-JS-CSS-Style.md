---
title: "[Frontend] CSS style"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [css, style]
render_with_liquid: false
future: true
---

## 📌 들어가며

HTML을 대충 마치고 이번엔 **CSS**를 살짝 배웠다. HTML로 만든 구조에 **스타일(디자인)**을 입히는 것이 CSS다.

---

## 1. 왜 CSS가 필요한가?

링크 3개에 색을 주고 싶다면 각각에 `style`을 넣으면 된다. 하지만 링크가 **수백~수천 개**인 대형 사이트라면? 일일이 다는 건 비효율적이다. 이때 **CSS로 한 번에** 스타일을 적용한다.

```html
<style>
    a {                        /* 모든 a 태그에 영향 */
        color: red;
        text-decoration: none; /* 밑줄 제거 */
    }
</style>
```

> 💡 `<style>` 안의 `a { }`는 **모든 `<a>` 태그에 영향**을 준다. 마치 상속과 같은 개념이다. HTML보다 훨씬 다양한 꾸밈 기능이 있다.

---

## 2. 개별 vs 일괄 스타일

```html
<ol>
    <li><a href="1.html">HTML</a></li>
    <!-- 개별 스타일: 이 링크만 파란색 밑줄 -->
    <li><a href="2.html" style="color:blue; text-decoration:underline;">CSS</a></li>
    <li><a href="3.html">JS</a></li>
</ol>
```

| 방법 | 적용 범위 |
|------|------|
| `<style> a { }` | **모든** a 태그 (일괄) |
| `<a style="...">` | **그 태그만** (개별) |

> 💡 개별 스타일은 일괄 스타일을 덮어쓴다. 적은 개수는 개별로, 많은 개수는 `<style>`로 일괄 처리하는 것이 효율적이다.

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/CSS-Style/1.png)

---

## 3. 여백 조정

```html
<p style="margin-top: 40px;">위쪽 여백 40px</p>
```

`margin`으로 요소 바깥의 여백을 조정할 수 있다.

---

## 📝 정리

```
CSS 기초
├─ 목적    HTML 구조에 디자인 입히기
├─ 일괄    <style> a { color: red; } → 모든 a 태그
├─ 개별    <a style="..."> → 그 태그만 (우선)
└─ 속성    color, text-decoration, margin ...
```

| 개념 | 한 줄 정의 |
|------|------|
| **CSS** | HTML의 디자인·레이아웃 담당 |
| **`<style> 태그{}`** | 태그별 일괄 스타일 |
| **인라인 style** | 개별 요소 스타일(우선) |

CSS는 "구조(HTML)와 디자인(CSS)의 분리"라는 웹의 핵심 원칙을 실현한다. 태그 선택자로 한 번에 스타일을 적용하면, 대형 사이트도 효율적으로 꾸밀 수 있다.
