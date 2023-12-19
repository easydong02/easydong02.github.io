---
title: "[Java] 자바 메모장 코딩"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 우리가 당연하게 여겼던 도구 eclipse를 사용하지 않고 자바의 본연의 모습을 잘 볼 수 있는 메모장코딩을 해보겠습니다.

먼저 출력문을 한번 해볼까요?

```
class Test{
public static void main(String[] args){
   System.out.println("신동희");
    }
}
```

제 이름을 출력하는 출력문을 만들었습니다. 근데 이클립스에서 작성을 안 하고 메모장에서 했죠

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/1.png)

보이시나요? 자동 완성도 없이 직접 타이핑하는 느낌이란.. 항상 자전거타다가 걸어가는 느낌이 드는군요.

이렇게 만들었는데 도대체 어떻게 java로 실행을 할까요? 이클립스가 있으면 컴파일과 실행까지 다 해주지만 우리가 직접코딩하려면 저 둘을 따로따로 해주어야 합니다. 먼저 cmd를 켜봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/2.png)

먼저 저 메모장이 있는 폴더를 가고 주소를 누르면 위치가 쭉나옵니다. 저걸 복사한 다음,

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/3.png)

이렇게 cmd에 cd라는 키워드 뒤에 그대로 붙여주면 기본 이동 경로가 바뀝니다. 그런데 여전히 경로는 바뀌지 않았습니다. 왜냐하면 바꾼 경로가 d드라이브 이기 때문에 드라이브를 바꾸어 주어야 하기 때문이죠.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/4.png)

이렇게 d:를 입력하면 기본 이동경로가 저 메모장이 있는 폴더위치로 바뀝니다.

이제 경로를 찾았으니 저 메모장을 코딩해야겠죠? 먼저

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/5.png)

저 메모장을 java파일로 만들어야합니다. 따라서 다른 이름으로 저장을 하시고 파일 이름 뒤에 확장자 .java를 붙여줍시다. 그리고 형식은 모든파일로 바꿔주시고요. 이러면 저 메모장은 더이상 txt파일이 아닌 java파일이 됩니다. 그리고 이 java파일을 컴파일해줍시다.

cmd 창에서 우리 자바 처음 배울 때 버젼 체크할 때 javac -version 이라고 입력했죠? 이번에는 javac(자바컴파일) java파일 이름을 적어주시면 됩니다. 여기서 주의할 점은 확장자 명까지 적어주셔야 됩니다.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/6.png)

저렇게 입력을 하고 엔터를 누르면 그 폴더에

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/7.png)

저런 class확장자의 파일이 보이실겁니다. 저건 컴퓨터가 알아들을 수 있는 바이너리 코드로 작성된 파일이죠. 이제 이것을 jvm으로 실행시키면 되는 겁니다. 입력 명령어는 cmd에서 java 파일명 해주시면 됩니다. 이때는 컴파일 할 때와 다르게 확장자명까진 입력 안 해도 됩니다.

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/8.png)

보시면 출력문에 썼던 문자열 "신동희"가 잘 나오는 것을 확인할 수 있습니다. 전에는 몰랐는데 메모장 코딩을 하면서 자바가 실제로 어떻게 실행이 되는지를 그 메커니즘을 체감할 수 있어서 좋았습니다. 자바에 대해서 근본적인 부분을 자세히 느낄 때까진 이클립스를 안 쓰고 한번 온전히 자바를 느껴보겠습니다!

오늘 수업엔 여러가지 기본 문제들을 풀었는데요, 제가 헷갈린 것들을 골라서 한번 올려보겠습니다

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/9.png)

쉬운 줄알았는데 헷갈리는 부분이 있더라고요. (나)에서 int a =3; 을 볼 때는 당연히 안 되는 줄 알았지만 (다)에서 새로운 블럭 안에 있으니 에러가 안 날줄 알았습니다. 그런데 알고보니 처음 선언한 a가 그 블럭 안에서 전역변수처럼 있기 때문에 다른 블럭을 만들어도 선언되지 않겠죠? 그리고 다른 블럭에서의 a=5; 도 적용이 안 되는 줄 알았습니다. 하지만 처음 a를 제일 큰 블럭에서 선언하였기 때문에 적용이 된다는 사실! 이클립스에서 적었으면 에러 내용도 다 볼 수 있을텐데 없이 하려니까 제가 생각하게 되더라고요 ㅎㅎ

![Desktop View](/assets/img/Programming-Language/Java/Notepad-Coding/10.png)

여기서 (마) 부분을 볼게요. 겉보기엔 문제가 없을 것 같지만 문제가 있습니다. 자바의 정수의 기본형은 int죠? 왜냐하면 최적화된 자료형이기 때문입니다. 따라서 int형보다 하위단계의 자료형들은 연산을 했을 때 int 형으로 바꿔주기 때문에 sum이라는 변수도 int형으로 해야합니다. 하지만 반대로 long 타입끼리의 연산은 int형으로 자동형변환을 하지 않습니다. 큰 단위에서 작은 단위로 내려오면 데이터 손실이 있거든요.

오늘 JAVA시간엔 여러가지 들을 복습하며 다지는 시간이 되어서 좋았습니다. 남은 시간엔 데이터베이스를 배워서 그것도 올릴게요~