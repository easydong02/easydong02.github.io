---
title: "[Git] 인텔리제이 터미널 git bash변경"
date: 2024-02-05 00:00:00 +0900
categories: [Git]
tags: [git, intellj]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 IntelliJ 내장 터미널을 **PowerShell에서 Git Bash로 변경**하는 방법을 정리한다. 윈도우 기본값인 PowerShell에서는 `git stash drop` 같은 일부 명령이 제대로 동작하지 않아, 터미널 셸을 바꾸는 것이 편하다.

> **왜 바꾸나?** IntelliJ 터미널의 윈도우 기본값은 **PowerShell**인데, 여기서는 일부 git 명령(`git stash drop` 등)이 안 먹히는 경우가 있다. **Git Bash**로 바꾸면 리눅스 스타일 셸에서 git 명령을 온전히 쓸 수 있다.

---

## 1. 현재 셸 확인

터미널을 열면 `powershell.exe`로 되어 있는 것을 볼 수 있다.

![Untitled](/assets/img/Git/Terminal/Untitled.png)

---

## 2. 설정 진입

`File → Settings`(윈도우 단축키 **Ctrl + Alt + S**) → `Tools → Terminal`로 이동한다.

![Untitled](/assets/img/Git/Terminal/Untitled%201.png)

---

## 3. Shell path를 Git Bash로 변경

Application Settings의 **Shell path**를 Git 설치 폴더의 `bin/bash.exe`로 지정하고, 옵션에 `-login -i`를 준다.

```
<Git 설치 경로>\bin\bash.exe -login -i
```

![Untitled](/assets/img/Git/Terminal/Untitled%202.png)

> 💡 **`-login -i` 옵션**은 로그인 셸(`-login`)로 대화형(`-i`) 실행하라는 뜻이다. 이러면 기본 로그인 정보로 셸이 시작되어, 프로필 설정(alias·환경 변수)까지 적용된 정상적인 Git Bash 환경이 열린다.

---

## 4. 새 터미널 확인

설정 후 터미널을 새로 추가하면, 그때부터 **Git Bash**가 실행된다.

![Untitled](/assets/img/Git/Terminal/Untitled%203.png)

---

## 📝 정리

```
IntelliJ 터미널 → Git Bash
├─ 이유   PowerShell에서 일부 git 명령 오류
├─ 경로   Settings → Tools → Terminal
├─ 설정   Shell path = ...\bin\bash.exe  옵션 -login -i
└─ 확인   새 터미널 열면 Git Bash 실행
```

| 항목 | 값 |
|------|------|
| **단축키** | Ctrl + Alt + S |
| **Shell path** | `bin\bash.exe` |
| **옵션** | `-login -i` |

IntelliJ에서 git 명령이 이상하게 동작한다면, 터미널 셸을 **Git Bash로 바꾸는 것**이 가장 간단한 해결책이다. `-login -i` 옵션만 잊지 않으면 익숙한 Bash 환경을 그대로 쓸 수 있다.
