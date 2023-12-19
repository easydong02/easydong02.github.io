---
title: "[C] 표준 입출력 헤더와 출력"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

비쥬얼 스튜디오에서 기본적으로 어떤 프로그래밍을 하기 위해선 기본적인 '틀'을 짜야한다고 합니다.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/1.png){: width="100%" }

이런식으로 #include <stdio.h> <ㅡㅡ이것을 표준 입출력(standard input/output) 헤더 (header)라고 합니다.

아직은 자세히 헤더에 대해서 배우진 않았지만 배운다면 거기에 대해서 추후 포스팅을 하려고 합니다.

밑에 보이는 main()은 함수라고 하는데요, C프로그램의 시작점이라고 합니다. 그리고 나오는 { 가 블럭의 시작인데 여기서 컴파일러는 개발자가 적어 놓은 코드를 읽게 되는거라고 보시면 됩니다. 우리가 적은 문장은 기본적으로 위에서 아래로, 왼쪽에서 오른쪽으로 읽습니다. 따라서 나중에 코드가 많이 복잡해질 때에는 우리가 컴파일러가 되어서 차근차근 읽어보는 방법도 좋다고 합니다.

**출력**

출력은 보통 'printf'라는 것으로 출력을 한다고 합니다. 기본적인 방식은

printf("출력하고 싶은 내용"); 으로 출력할 내용(=문자열)은 반드시 쌍따옴표 쌍으로 묶어주고 하나의 문장(statement)에서 문장의 끝은 반드시 ;(세미콜론)으로 끝나야 합니다.

그렇다면 제가 '동희의 IT 일기장'이라는 문구를 한번 출력해보도록 하겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/2.png){: width="100%" }

이런식으로 쌍따옴표 안에 제가 '동희의 IT 일기장'이라는 문자열을 삽입하였는데요, 출력이 되는지 빌드를 해보겠습니다! 빌드는 Ctrl+F5로 할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Common-IO-Header/3.png){: width="100%" }

창을 캡쳐하니까 글씨가 좀 작네요.. 하지만 보시는 바와 같이 정상적으로 제가 원하는 것을 출력되는게 보입니다.

그리고 main()함수가 끝나면 프로그램은 종료됩니다. 이번시간에는 제가 배운 출력에 대해서 써봤는데요, 앞으로 배울 것이 많겠지만 하나씩 차근차근 해보도록 하겠습니다.
