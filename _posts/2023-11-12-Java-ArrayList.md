---
title: "[Java] ArrayList"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **ArrayList**를 활용해 간단한 **회원가입·로그인** 응용 문제를 풀어본다. 아이디 중복 검사, 비밀번호 암호화까지 넣어, 배운 것들을 하나로 엮어보자.

> **왜 배열이 아니라 ArrayList?**
> 일반 배열은 크기가 고정이지만, `ArrayList`는 **추가·수정·삭제가 자유롭다.** 회원이 늘고 주는 상황에 적합하다.

**전체 구조**는 세 클래스로 나눈다.

| 클래스 | 역할 |
|------|------|
| `User` | 회원 데이터(id, pw)를 담는 객체 |
| `UserField` | DB(ArrayList) + 중복검사·암호화·회원가입·로그인 로직 |
| `UserMain` | 실제로 실행하는 진입점 |

---

## 1. User 클래스 — 데이터 객체

id와 pw만 두고, `private` 필드에 접근할 수 있도록 기본 생성자와 Getter/Setter를 만든다.

```java
package day16;

public class User {
    private String id;
    private String pw;

    public User() {}

    public String getId()          { return id; }
    public void   setId(String id) { this.id = id; }
    public String getPw()          { return pw; }
    public void   setPw(String pw) { this.pw = pw; }
}
```

---

## 2. UserField 클래스 — 로직 모음

### DB (ArrayList)

```java
ArrayList<User> users = new ArrayList<>();   // User 객체들을 저장
Scanner sc = new Scanner(System.in);
```

### 아이디 중복검사

```java
public User checkId(String id) {
    User user = null;
    for (int i = 0; i < users.size(); i++) {
        if (users.get(i).getId().equals(id)) {
            user = users.get(i);   // 중복이면 그 객체 반환
        }
    }
    return user;                   // 중복 없으면 null 반환
}
```

> 💡 중복이면 해당 `User` 객체를, 없으면 `null`을 반환한다. 이 반환값을 로그인·회원가입 양쪽에서 활용한다.

### 암호화 (단방향)

```java
public String encrypt(String pw) {
    String en_pw = "";
    for (int i = 0; i < pw.length(); i++) {
        en_pw += (char)pw.charAt(i) * 3;   // 아스키코드 × 3
    }
    return en_pw;
}
```

> ⚠️ pw를 평문으로 저장하면 보안 위험이 있어, 간단히 **아스키코드 × 3**으로 단방향 암호화했다. (실무에서는 BCrypt 등 검증된 해시를 써야 한다.)

### 회원가입

```java
public void makeId(User user) {
    user.setPw(encrypt(user.getPw()));   // 암호화한 pw로 교체
    users.add(user);                     // ArrayList에 추가
}
```

> 💡 **하나의 메소드에 모든 기능을 몰아넣지 말자.** 중복검사는 별도 메소드로 빼고, 회원가입은 "암호화 + 추가"만 담당하게 했다.

### 로그인

```java
public boolean logIn(String id, String pw) {
    User user = checkId(id);                       // 아이디 존재 확인
    if (user != null) {                            // 있어야 로그인 가능
        if (user.getPw().equals(encrypt(pw))) {    // 암호화 후 비교
            return true;
        }
    }
    return false;
}
```

---

## 3. UserMain 클래스 — 실행

```java
package day16;

public class UserMain {
    public static void main(String[] args) {
        UserField uf = new UserField();
        User newUser = new User();

        User checkUser = uf.checkId("easydong02");   // ① 중복 검사

        if (checkUser == null) {                     // 없으면 가입
            System.out.println("사용 가능한 아이디");
            newUser.setId("easydong02");
            newUser.setPw("1234");
            uf.makeId(newUser);
            System.out.println("회원가입 성공!");
        } else {
            System.out.println("중복된 아이디");
        }

        checkUser = uf.checkId("easydong02");        // ② 같은 아이디 재검사

        if (checkUser == null) {
            System.out.println("사용 가능한 아이디");
        } else {
            System.out.println("중복된 아이디");     // 이번엔 중복!
        }
    }
}
```

동작 흐름은 이렇다.

```
① easydong02 검사 → 없음 → 가입 → "사용 가능한 아이디" + "회원가입 성공!"
② easydong02 재검사 → 이미 있음 → "중복된 아이디"
```

![Desktop View](/assets/img/Programming-Language/Java/ArrayList/1.png)

출력 결과: **"사용 가능한 아이디" → "회원가입 성공!" → "중복된 아이디"** 세 문장이 잘 나온다.

---

## 📝 정리

```
ArrayList 회원 관리
├─ User       데이터 객체 (id, pw + Getter/Setter)
├─ UserField  DB(ArrayList) + checkId · encrypt · makeId · logIn
└─ UserMain   실행: 가입 → 재검사로 중복 확인
```

| 개념 | 한 줄 정의 |
|------|------|
| **ArrayList** | 크기가 유동적인 리스트 (add/get/size) |
| **역할 분리** | 한 메소드가 한 가지 일만 하도록 |
| **단방향 암호화** | 복원 불가능하게 pw를 변형 |

회원가입·로그인이 처음엔 어려워 보였지만, 배운 것들(클래스·ArrayList·메소드·조건문)을 차근차근 조합하니 충분히 만들 수 있었다. 실무에서는 본인인증·비밀번호 규칙 등이 더해지지만, 기본 골격은 이와 같다.
