---
title: "[C] 함수"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

함수란?

함수 (function)

자주 반복되어 수행할 코드 덩어리(?) 들을

1\. 함수(function)로 '정의' 하고

2\. 필요할때마다 정의된 함수를 '호출'하여 사용하는겁니다.

1.함수 정의(function definition) 형식

리턴타입 함수이름(매개변수들 ..)

{

함수 본체 ( 수행 코드들)

}

\*여기서 우리는 함수의 원형(function prototype)을 알아봅시다.

\= 리턴타입 함수이름(매개변수들) 이며 앞으로 함수의 선언부분에서 다시 설명하는데 함수의 선언만 할 때에는 함수의 원형을 씁니다. 그리고 visual studio 의 소스파일에 각 각 어떤 함수가 쓰였는데 함수의 원형의 형식으로 적혀있죠!

![Desktop View](/assets/img/Programming-Language/C/Function/1.png){: width="100%" }

이렇게 소스파일 이름 왼쪽에 삼각형 모양의 버튼이 있어서 그 소스파일에 어떤 함수가 쓰여있는지 열고 닫으면서 볼 수 있습니다!

\*함수의 호출(function call, function invocation)의 형식

함수이름(매개변수에 본인이 원하는 변수값 등을 넣어도 된다);

의 식으로 호출합니다.

그래서 우리가 소스파일을 만들때 항상 했던 main()의 경우도 함수의 일종이라고 할 수 있습니다.

\*함수 알고리즘: 함수는 일단 호출이 되는 코드문을 컴파일러가 읽으면 거기서 잠깐 멈추고 호출된 함수를 실행시킵니다. (TMI: 호출한 함수: caller 호출된 함수: callee라고 합니다.) 호출된 함수를 다 읽으면 그다음에 다시 호출한 함수를 멈춘 부분부터 다시 읽습니다. 이를 리턴이라고 합니다.

\*함수 매개변수(parameter), 함수 인자(argument)

매개변수는 함수 정의할때 명시하는 변수 (그 함수의 지역변수로서 동작합니다)

인자 는 함수 호출할때 입력하여 매개변수에 전달하는 값

함수 호출시 호출인자가 매개변수에 '복사' 되어 호출한 함수 수행된다고 합니다!

※ 두 용어는 혼용되어 사용되곤 합니다.

그렇다면 글만 있으면 지루하니까 예시를 보여드리겠습니다.

oper라는 이름의 함수에 매개변수 int 타입의 정수 2개를 설정하고 리턴타입은 void로 하겠습니다. 여기서 void란 리턴값이 없다는 뜻입니다. 그리고 본체엔 int 타입의 k,v를 받아서 사칙연산을 해보도록 하겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Function/2.png){: width="100%" }

이렇게 원래 있던 main함수 바깥에 새로운 함수를 형성했습니다. 하지만 이렇게 실행을 하면 더 oper함수 안에있는 문장들이 실행 될까요? 아닙니다. 위에 말씀드린것처럼 main함수에서 저 함수를 호출해야하는데요,

![Desktop View](/assets/img/Programming-Language/C/Function/3.png){: width="100%" }

위에 언급한대로 호출을 해봤습니다. 이 문장의 뜻은 oper 함수에 k에는 100, v에는 10을 넣어서 실행한다는 뜻입니다.

![Desktop View](/assets/img/Programming-Language/C/Function/4.png){: width="100%" }

정상적으로 작동을 하네요 ^^

\*\*주의사항

함수를 만들고 이용할 때에는 방금같은 경우에선 함수가 한개만 단순히 사용되었는데요, 실무에서는 다양한 함수들이 서로 서로 안에 들어가 있고 복잡해집니다. 그래서 함수를 이용할 때에는 그 순서도 잘 고려해야하는데요, 예를 들어 aaa()함수와 bbb()함수를 각각 만들었다고 합니다. main()함수안에서 aaa()함수를 호출시키는데 그 aaa()함수안에 bbb()함수를 호출시키는 문장이 있다면 어떻게 될까요?

존재하지 않는 이미지입니다.

이런 식으로 호출을 하고 또 호출을하는 순서가 이어지게 됩니다! 따라서 함수의 논리적인 사용순서가 중요하겠죠?

\*리턴(return)

자 그럼 함수의 작동에 대해선 알고있는데 그럼 리턴은 무엇인가요?

자기를 호출한 함수에 최종값 1개를 주는 것을 리턴값이라고 합니다. 매개변수에 원하는 값을 넣어서 함수의 식을 작동하고 나온 결과값이라고 생각하시면 됩니다.

![Desktop View](/assets/img/Programming-Language/C/Function/5.png){: width="100%" }

함수 2개를 만들었습니다. 첫번째는 sum이라는 지역변수를 만들고 매개변수 둘을 더한 값을 그 sum에 대입시키고 그 값을 리턴값에 넣어 호출한 함수로 돌려주는 형식입니다. add(10,5); 를 한다면 그 리턴값은 15가 되겠죠?

그렇다면 main()함수에 int result=add(10,5); 를 한다면 add()함수의 리턴 값은 15기 때문에 result라는 변수에는 15의 값이 대입이 되겠습니다.

두번째는 변수를 따로 만들지 않고 return에 식자체를 입력한 것인데요. 두번때 첫번째 매개변수에 두번째 매개변수를 빼는 식 자체를 return 값에 넣었습니다. 결과를 산출하는 방식은 차이가 없습니다~

다만 void 타입의 경우 구체적인 리턴값이 없으니 printf()와 같은 문장을 넣어서 호출하면 되겠죠?

\*\*순서!

우리가 변수를 배울 때에는 선언을 먼저하고 그 값을 이용한다고 했었죠? 함수도 마찬가지입니다. 먼저 어떤 함수인지 main()함수 들어가기 전에 '정의'를 해놔야 호출을 할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Function/6.png){: width="100%" }

이런식으로 main()함수 밑에 half라는 함수를 정의하면 실행할때 오류가 납니다. 왜냐하면 컴파일러는 위에서 아래로 읽기 때문이죠. 하지만 방법이 있습니다. '정의'는 밑에서 해도 '선언'만 위에서 해주면 되거든요!

함수의 선언은 함수의 원형으로 합니다. 위에서 설명했죠? 리턴타입 함수이름(매개변수)만 위에서 '선언'하면 '정의'를 밑에서 하더라도 이용할 수 있습니다!

![Desktop View](/assets/img/Programming-Language/C/Function/7.png){: width="100%" }

이런 식으로 main()함수 들어가기전에 선언만 해주었습니다. 정의는 main()함수 밑에 있죠! (double 타입으로 정의해서 main함수에도 double로 바꾸어 주었습니다.. 무의식적으로 int를 치는 습관...ㅜ) 선언을 하면 이제 실행이 될까요?

![Desktop View](/assets/img/Programming-Language/C/Function/8.png){: width="100%" }

잘 실행이 됩니다 ^^ 신기하네요!

자, 그럼 생각해봅시다. 하나의 프로젝트에 여러 소스파일이 있는데 그 소스파일마다 다 각각 함수를 선언하고, 정의하고 번거롭게 작업을 진행해야 할까요? 그렇다면 프로그래밍의 의미가 없어질 것입니다. 효율과 반복을 피하기 위해 사람들은 여러가지 도구를 개발하거거든요..

지금까지는 소스파일만 만들었지만 헤더 파일이란 것을 만들어 보면

![Desktop View](/assets/img/Programming-Language/C/Function/9.png){: width="100%" }

이런 식으로 만들었습니다. 헤더파일에는 함수의 선언, 매크로, 상수 등을 모아놓은 것인데요. 왜 함수의 선언을 모아두냐? 우리가 처음 소스파일을 만들면 제일 처음에 쓰는 문장이 무엇인가요?

보통 #include <stdio.h>라는 것입니다. 이것 또한 헤더파일인데요. 우리가 쓰는 기본적인 함수들이 여기에 있기 때문에 저걸 써야 우리가 그 함수들을 사용할 수 있게 되는 거지요. 마찬가지로 기본 헤더파일 말고도 우리가 위의 사진처럼 만들 수가 있습니다! 그리고 그것은 #include "헤더파일이름.h"으로 사용하죠. 따라서 선언부분은 main()위에서 했잖아요? 그래서 맨위에서 작동하는 헤더파일이기 때문에 여기에 선언을 정리해서 넣어두면 굳이 소스파일에 main()함수 위에 덕지덕지 함수들을 선언할 필요가 없어지죠!

제 프로젝트의 소스파일 목록과 헤더파일을 보여드릴게요

![Desktop View](/assets/img/Programming-Language/C/Function/10.png){: width="100%" }
이렇게 되어있습니다. 소스파일엔 여러가지가 있는데 금지표시(?)가 된 소스파일들은 빌드를 할 때 제외를 시켜서 서로 충돌되지 않도록 하였습니다. 그런데 이 말대로라면 하나빼고 다 빌드에서 제외를 해야하는데 왜 2개가 활성화가 되어있을까요? 하나는 main()함수를 포함하고 있고 하나는 함수의 '정의'들을 모아 놓은 소스파일입니다. 따라서 거기엔 main()함수가 없죠! 그렇다면 myfunction.c라는 소스파일을 볼까요?

![Desktop View](/assets/img/Programming-Language/C/Function/11.png){: width="100%" }

위에 #include <stdio.h>라는 문장이있고 그 다음이 이 사진처럼 있는게 다입니다. 따라서 이렇게 함수를 정의 해놓은 것을 모아두고 main()함수가 있는 소스파일, 즉 test.c에서는 따로 또 정의를 할 필요가 없이 호출만 하면 된다는 것이죠.

정리를 해보자면 한 소스파일에 함수를 '정의'하고, 헤더파일엔 함수를 '선언'하고, main()함수가 포함되어있는 소스파일엔 '선언'들이 모아져있는 헤더파일을 포함하겠다는 #include 문장을 넣어준다면, 그 소스파일에서는 그냥 '호출'만 하면 된다는 것입니다.

예시로 가보죠, 함수가 정의가 모여있는 소스파일을 보면 power라는 함수가 있죠? 이것은 간단하게 말하면 제곱을 해주는 함수를 만들었습니다. power(3,4)라고 하면 result 라는 변수명에 3의 4제곱, 즉 81이 리턴값으로 된다는 거죠. 그리고 위에 올린 헤더파일 사진을 보시면 power함수의 선언만 달랑 있습니다. 그러면 이것을 이용해서 main()함수가 있는 test.c 소스파일에서 호출만 해보도록할게요!

![Desktop View](/assets/img/Programming-Language/C/Function/12.png){: width="100%" }

include 문장으로 선언문이 있는 헤더파일을 포함한다고했으니까 선언은 되어있겠죠?

그리고 myfunction.c라는 파일엔 함수의 정의만 모아져있습니다. 그래서 main()함수에는 선언도, 정의도 없습니다. power(3,4)라는 값만 출력해보도록 하겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Function/13.png){: width="100%" }

오오...!! 신기하네요 ㅎㅎ 알면 알수록 재밌는 컴퓨터의 세계..

재귀호출(recursive call)

\- 함수가 실행중에 자신을 다시 호출하는 것입니다.

\- 복잡한 문제를 단순화하여 구현하는 장점이 있습니다.

\- 무한한 재귀호출은 불가합니다.

쉽게 말해서 어떤 함수를 호출하는데 그 함수안에 본인이 포함되어있다는거죠. 말로 하면 어려우니 보여드리겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Function/14.png){: width="100%" }

func1이란 함수를 정의했습니다. 그리고 그 리턴값은 또 func(n+1)이라네요.. 그럼 main()함수에서 func1()에 1을 대입해서 출력을하면 어떻게 될까요? 네 맞습니다. 무한히 출력되겠지요.

![Desktop View](/assets/img/Programming-Language/C/Function/15.png){: width="100%" }

계속 출력이 되고 있습니다. 따라서 그 함수 안에 if(n==?)라는 조건문을 넣고 이때 return값은 얼마로 하겠다 라는 문장을 넣어서 무한히 호출되는것을 막아야 합니다.

이번 시간에는 함수에 대해서 알아봤는데요. 어렵지만 알고보면 그만큼 재밌는 부분이라 생각합니다!
