---
title: "[Git] 인텔리제이 터미널 git bash변경"
date: 2024-02-05 00:00:00 +0900
categories: [Git]
tags: [git, intellj]
render_with_liquid: false
---

# 인텔리제이 터미널 - Git bash로 설정해보자

인텔리제이에서 터미널에서 git 관련 명령어를 할 때가 많습니다. 윈도우 기준 기본 설정인 poweshell로 터미널이 되어있는데 이 경우 git stash drop 등 안 먹히는 명령어들이 많았습니다. 그래서 이번엔 인텔리제이에서 터미널 툴을 바꿔보겠습니다.

## ✅ 인텔리제이 설정 변경

---

![Untitled](/assets/img/Git/Terminal/Untitled.png)

→ 인텔리제이 터미널을 보면 이렇게 powershell.exe로 되어있음을 알 수 있습니다.

![Untitled](/assets/img/Git/Terminal/Untitled%201.png)

→ File - Settings(윈도우 단축키: Ctrl + Alt + S)로 들어갑니다. 그리고 Tools - Terminal에 들어갑니다

![Untitled](/assets/img/Git/Terminal/Untitled%202.png)

→ Application Settings에서 Shell path를 Git이 설치되어 있는 폴더에 bin 폴더에서 bash.exe를 선택하고 옵션으로 -login -i로 합니다. 그러면 기본 로그인 정보로 로그인을 합니다.

![Untitled](/assets/img/Git/Terminal/Untitled%203.png)

→ 그리고 터미널을 새로 추가하면 그때부터는 git bash가 실행됩니다!