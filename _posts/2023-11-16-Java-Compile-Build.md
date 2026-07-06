---
title: "[Java] 수동 컴파일, 수동 빌드 실행"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

Eclipse를 쓰지 않으면, 다른 패키지의 클래스를 import하려면 **모든 걸 수동으로 제어**해야 한다. 이번 글에서는 EditPlus로 패키지를 구축하고, 다른 패키지의 클래스를 import·컴파일·실행하는 과정을 정리한다.

> **목표 구조**: `src` 폴더에 `.java`, `bin` 폴더에 `.class`를 분리해서 넣는다.

```
testapp/
├─ src/   ← .java 파일
└─ bin/   ← .class 파일 (컴파일 결과)
```

---

## 1. 환경 변수(classpath) 설정

`.class` 파일이 생성될 기준 경로를 **classpath 환경 변수**로 지정한다.

```
내 PC 우클릭 → 속성 → 고급 시스템 설정 → 환경 변수
→ 시스템 변수 새로 만들기
   변수 이름: classpath
   변수 값:   ...\testapp\bin
```

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/3.png)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/5.png)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/6.png)

> 💡 `bin`까지 classpath로 지정하면, 코드에서는 `com.koreait.tool` 같은 **패키지 경로만** 쓰면 되고 전체 경로를 반복할 필요가 없다.

---

## 2. 파일 작성 — Pen과 UsePen

`Pen.java`(메인 없음)와, 다른 패키지에서 이를 사용하는 `UsePen.java`를 만든다.

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/8.png)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/10.png)

> 💡 `import` 경로가 `com`부터 시작하는 이유: classpath로 `bin`까지 지정했고, 그 뒤 경로(`com.koreait...`)는 java 파일 안의 패키지 선언에 있기 때문이다.

---

## 3. 수동 컴파일 (`javac -d`)

`Pen.java`가 있는 폴더로 이동한 뒤, **`-d` 옵션**으로 class 생성 경로를 지정해 컴파일한다.

```bash
cd ...\testapp\src\...\tool          # Pen.java가 있는 폴더
javac -d D:\bigData\...\testapp\bin Pen.java
```

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/11.png)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/12.png)

> 💡 **`-d 경로`**는 같은 폴더에 class를 만드는 것을 막고, **지정 경로에 패키지 구조대로** class를 생성한다. `com` 폴더가 없어도 알아서 만들어준다. `Pen.class`는 메인이 없어 실행 불가이고, 실행은 이를 사용하는 `UsePen`에서 한다.

같은 방식으로 `UsePen.java`도 컴파일한다.

---

## 4. 수동 실행 (`java`)

```bash
java com.koreait.usetool.UsePen     # 확장자 없이 '패키지경로.클래스명'
```

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/13.png)

| 단계 | 명령 | 특징 |
|------|------|------|
| 컴파일 | `javac -d bin경로 파일.java` | 확장자 필요, 경로 지정 |
| 실행 | `java 패키지.클래스명` | 확장자 없이, 패키지 경로로 |

---

## 5. EditPlus 단축키로 자동화

매번 cmd로 하기 번거로우니, EditPlus의 **사용자 도구**로 등록한다. (`도구 → 사용자 도구 구성`)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/14.png)

![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/15.png)

| 명령 | 실행 파일 | 인수 |
|------|------|------|
| **컴파일(Ctrl+1)** | `javac` | `-d D:\...\bin $(FileName)` |
| **실행(Ctrl+2)** | `java.exe` | `$(Prompt) $(FileNameNoExt)` |

> 💡 실행 시 `com.koreait.usetool`은 코드에 있는 경로라 EditPlus가 자동 추론 못 한다. 그래서 **`인수 묻기($(Prompt))`**를 넣어, 실행할 때 프롬프트에 패키지 경로를 직접 입력한다. 파일명은 확장자 없는 **`$(FileNameNoExt)`**를 쓴다.

---

## 📝 정리

```
수동 컴파일·실행
├─ 구조     src(.java) / bin(.class) 분리
├─ classpath  bin을 환경 변수로 등록
├─ 컴파일   javac -d bin경로 파일.java
├─ 실행     java 패키지경로.클래스명 (확장자 X)
└─ 자동화   EditPlus 사용자 도구 (Ctrl+1 컴파일, Ctrl+2 실행)
```

| 개념 | 한 줄 정의 |
|------|------|
| **classpath** | class 파일의 기준 경로 |
| **javac -d** | class를 지정 경로에 패키지 구조로 생성 |
| **java 패키지.클래스** | 패키지 경로로 실행 |

IDE 없이 컴파일·실행을 해보니, Eclipse가 감춰주던 **패키지·classpath·컴파일 경로**의 동작을 깊이 이해할 수 있었다. 이 원리를 알면 빌드 도구의 동작도 훨씬 명확하게 보인다.
