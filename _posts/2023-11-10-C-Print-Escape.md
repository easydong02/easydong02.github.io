---
title: "[C] 기본출력과 Escape문자"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

이번엔 기본출력과 문자열과 escape문자에 대해서 배운걸 써보겠습니다.

printf 의 뜻은 print formatted(서식화된 출력)입니다. 그리고 쌍따옴표 안에는 문자열이 있습니다.

"문자열"출력

printf("Hello World\\n");

위 문장은 안에 Hello World를 출력한다는 뜻

문자열(string) 선언에서는, 예를 들어,

최대 20개의 문자를 담을 수 있는 문자열 str1 선언

char str1\[20\]="C Language"; 라고 쓴다면 str1이라는 문자변수에 20개의 문자를 담을 수 있습니다.

그리고 C Language는 총 10문자, 공백도 하나의 문자!

printf("str1=%s\\n", str1);로 출력을 하게 된다면,

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/1.png){: width="100%" }

이런식으로 잘 나오네요!

char str2\[\] = "Java Language"; 이런 식으로 비워 놓게 되면 자동적으로 초기화, 문자열의 문자갯수만큼의 크기로 생성합니다.

printf("str2=%s\\n", str2);로 역시 출력해봅시다.

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/2.png){: width="100%" }

오호!

char \*str3 = "Python Language"; \*는 포인터 사용을 한 것인데 아직 자세히는 안 배웠습니다..

printf("str3=%s\\n", str3);

escape 문자에 대해서 봅시다. \\n 같이 줄바꿈 할 때 쓰는 문자도 escape문자인데요.

이것은 문자열 안에서 ("~" 안에서) 사용하는 특수한 기능을 하는 문자입니다.

\\n : 줄바꿈

\\t : 탭: 8칸을 한탭

\\" : 쌍따옴표

\\\\ : \\출력

\\0 : 널(null)문자

이를 이용해서 (say "hi")라는 문구를 출력하고 싶습니다. 하지만 쌍따옴표는 그냥 쓸 수가 없기 때문에 escape 문자를 써서

printf("say \\"hi\\"\\n"); 이런식으로 해야합니다.

![Desktop View](/assets/img/Programming-Language/C/Print-Escape/3.png){: width="100%" }

잘 나왔네요 ^^

이번시간은 여기까지~
