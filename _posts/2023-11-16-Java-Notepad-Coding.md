---
title: "[Java] 자바 메모장 코딩"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

당연하게 여기던 도구 Eclipse를 잠시 내려놓고, **메모장(Notepad)만으로 자바를 코딩**해본다. 자동 완성도, 즉각적인 오류 표시도 없이 직접 컴파일하고 실행하면서, **자바가 실제로 어떻게 돌아가는지 그 메커니즘**을 체감하는 것이 목적이다.

---

## 1. 메모장으로 코드 작성

먼저 이름을 출력하는 간단한 코드를 메모장에 작성한다.

```java
class Test {
    public static void main(String[] args) {
        System.out.println("신동희");
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/1.png)

> 💡 자동 완성 없이 직접 타이핑하는 느낌은, 마치 자전거를 타다가 걸어가는 것 같다. 그만큼 IDE가 많은 걸 대신해주고 있었다는 뜻이다.

---

## 2. 컴파일 → 실행 (수동)

Eclipse는 컴파일과 실행을 한 번에 해주지만, 직접 하려면 **두 단계를 따로** 진행해야 한다.

```
.java (소스)  ──javac──►  .class (바이트코드)  ──java──►  실행(JVM)
              컴파일                          실행
```

### ① cmd에서 경로 이동

메모장이 있는 폴더 주소를 복사해 `cd`로 이동한다. 단, **드라이브가 바뀌면**(예: D 드라이브) `cd`만으로는 안 되고 `d:`를 입력해 드라이브부터 바꿔야 한다.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/2.png)

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/3.png)

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/4.png)

### ② .java 파일로 저장

메모장을 **다른 이름으로 저장 → 확장자 `.java` → 형식을 '모든 파일'**로 바꿔 저장한다. 이제 txt가 아니라 java 파일이다.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/5.png)

### ③ 컴파일 (`javac`)

| 명령 | 설명 | 확장자 |
|------|------|:---:|
| `javac 파일명.java` | 컴파일 → `.class` 생성 | **필요** |
| `java 파일명` | JVM으로 실행 | **불필요** |

```bash
javac Test.java   # 컴파일 (확장자까지)
```

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/6.png)

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/7.png)

`.class` 파일이 생성된다. 컴퓨터가 알아듣는 바이너리(바이트코드) 파일이다.

### ④ 실행 (`java`)

```bash
java Test   # 실행 (확장자 없이)
```

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/8.png)

출력문에 쓴 `신동희`가 잘 나온다. 메모장 코딩을 통해 **자바의 컴파일→실행 메커니즘**을 직접 체감할 수 있었다.

---

## 3. 오늘의 헷갈림 정리

### 변수 스코프

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/9.png)

> 💡 큰 블록에서 선언한 `int a = 3;`은, 그 안에 새 블록을 만들어도 **여전히 유효**하다. 그래서 안쪽 블록에서 다시 `int a`로 선언하면 중복 오류가 나고, 안쪽 블록의 `a = 5;`는 바깥의 `a`에 **적용된다**. Eclipse 없이 하니 오류 메시지가 안 보여 스스로 더 생각하게 됐다.

### 정수 연산의 기본형은 int

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/10.png)

> ⚠️ 자바에서 정수의 기본형은 **`int`**(최적화된 자료형)다. 그래서 `int`보다 하위 자료형끼리 연산하면 결과가 **`int`로 자동 형변환**된다. 따라서 결과를 담는 변수도 `int`여야 한다. 단, **`long`끼리의 연산은 `int`로 내려가지 않는다** — 큰 단위에서 작은 단위로 내려가면 데이터 손실이 생기기 때문이다.

---

## 📝 정리

```
메모장 자바 코딩
├─ 작성    class { main } 을 .java로 저장
├─ 컴파일  javac 파일명.java  → .class (바이트코드)
├─ 실행    java 파일명        → JVM 실행
└─ 배운점  변수 스코프 / 정수 연산 기본형은 int
```

메모장 코딩은 비효율적이지만, IDE가 감춰주던 **컴파일과 실행의 본질**을 드러내준다. 자바의 근본을 온전히 느낄 때까지는 한동안 이 방식으로도 연습해보려 한다.
