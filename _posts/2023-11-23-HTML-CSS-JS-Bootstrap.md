---
title: "[Frontend] 부트스트랩"
date: 2023-11-23 00:00:00 +0900
categories: [Frontend, HTML-CSS-JS]
tags: [html, bootstrap]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **Bootstrap**의 무료 템플릿을 가져와 내 웹페이지를 꾸며본다.

> **Bootstrap이란?** 미리 만들어진 CSS·JS 컴포넌트(네비게이션 바, 버튼, 그리드 등)를 제공하는 프론트엔드 프레임워크. 복잡한 CSS를 직접 짜지 않고도 세련된 UI를 빠르게 만들 수 있다.

---

## 1. 템플릿 가져오기

Bootstrap 홈페이지의 **Examples**에서 원하는 예시(여기선 **Navbar static**)를 고른다.

```
Bootstrap Examples → Navbar static 선택
→ 페이지 우클릭 → 페이지 소스 보기 → HTML 복사
```

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/Bootstrap/2.png)
![Desktop View](/assets/img/Frontend/HTML-CSS-JS/Bootstrap/3.png)

> 💡 Bootstrap을 쓰려면 `<head>`에 **Bootstrap CDN 링크**(CSS·JS)를 포함하면 된다.

```html
<!-- Bootstrap core CSS -->
<link rel="stylesheet" href="https://.../bootstrap.min.css">
<script src="https://.../bootstrap.min.js"></script>
```

---

## 2. 네비게이션 바 커스터마이징

복사한 템플릿에서 **class**만으로 스타일이 적용된다. 브랜드명과 메뉴만 내 내용으로 바꾼다.

```html
<nav class="navbar navbar-expand-md navbar-dark bg-dark mb-4">
  <div class="container-fluid">
    <a class="navbar-brand" href="index.html">Easydong02</a>  <!-- 브랜드명 -->
    ...
    <ul class="navbar-nav me-auto mb-2 mb-md-0">
      <li class="nav-item"><a class="nav-link active" href="about.html">소개</a></li>
      <li class="nav-item"><a class="nav-link" href="project.html">프로젝트</a></li>
      <li class="nav-item"><a class="nav-link disabled">관리자</a></li>
    </ul>
  </div>
</nav>

<main class="container">
  <div class="bg-light p-5 rounded">
    <h1>프로젝트 목록</h1>
    <a class="btn btn-lg btn-primary" href="#" role="button">프로젝트 보기 »</a>
  </div>
</main>
```

주요 Bootstrap 클래스를 정리하면:

| 클래스 | 역할 |
|------|------|
| `navbar navbar-dark bg-dark` | 어두운 네비게이션 바 |
| `nav-link active` / `disabled` | 활성 / 비활성 링크 |
| `btn btn-lg btn-primary` | 크고 파란 버튼 |
| `container` / `p-5` / `rounded` | 컨테이너 / 패딩 / 둥근 모서리 |

![Desktop View](/assets/img/Frontend/HTML-CSS-JS/Bootstrap/4.png)

브랜드명이 바뀌고, 앞으로 여기에 프로젝트를 넣을 수 있는 골격이 완성됐다.

---

## 📝 정리

```
Bootstrap
├─ 목적    미리 만든 컴포넌트로 빠른 UI
├─ 사용    CDN 링크 포함 + class 지정
├─ 템플릿  Examples에서 소스 복사 후 커스터마이징
└─ 클래스  navbar / btn / container 등
```

| 개념 | 한 줄 정의 |
|------|------|
| **Bootstrap** | 프론트엔드 CSS·JS 프레임워크 |
| **CDN** | 라이브러리를 링크로 불러오기 |
| **class 기반** | 클래스만 붙이면 스타일 적용 |

Bootstrap은 "CSS를 직접 짜는 대신 검증된 클래스를 조합"하는 방식이다. 템플릿을 가져와 내용만 바꿔도 그럴듯한 페이지가 완성되니, 빠른 프로토타이핑에 특히 유용하다.
