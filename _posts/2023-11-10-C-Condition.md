---
title: "[C] 조건문"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

제어문 (Control) : 프로그램의 흐름을 제어(변경)하는 문장

1.조건문(Conditional)

: if ~ else, switch ~ case

2\. 순환문(loop)

: for, while, do ~ while

이 중 오늘은 조건문에 대해서 배웠습니다.

**1\. if ~ else 조건문**

if(조건식) <-- 조건식이 '참'이면

그 다음의 '한 문장' 혹은 '한 블럭' 을 실행시킵니다.

거짓이면 실행하지 않고 넘어간다.

![Desktop View](/assets/img/Programming-Language/C/Condition/1.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/2.png){: width="100%" }

이런식으로 출력이 잘 됩니다.

그런데 if만 쓴다면 재미가 없을 것입니다. 여기에 else if와 else를 설명드리겠습니다.

if(조건식1)

조건식1 이 '참' 일때 수행

else if (조건식2)

조건식2 가 '참' 일때 수행

else if (조건식3)

조건식3 가 '참' 일때 수행

..

else

\* 위 조건식들 모두 거짓일때 수행

이것처럼 esle if 는 위 문장이 거짓일때 수행하며 else if가 실행된다면 그 밑에 문장들은 실행시키지 않습니다.

그리고 마지막으로 else는 위에 모든 문장이 거짓일 때 보통 마지막으로 수행되는 문장입니다.

![Desktop View](/assets/img/Programming-Language/C/Condition/3.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/4.png){: width="100%" }

이것처럼 point에 무엇을 넣느냐에 따라서 뒤의 조건식 중 하나가 실행이 되는 겁니다!

만약 모든 if 가 포함된 문장에 해당되지 않는 경우, 마지막 else문장이 실행되겠죠?

**2\. switch 조건문**

보통 switch 문은

switch(정수)

case x :

break;

의 형태히고 case 오른쪽 x는 switch(정수) 에서 괄호 부분의 값을 나타냅니다.

그리고 각 케이스마다 break; 문장을 넣어서 작동을 멈추어 switch 블럭에서 빠져나오게 하는 문장입니다.

![Desktop View](/assets/img/Programming-Language/C/Condition/5.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Condition/6.png){: width="100%" }

n의 값을 1로 설정했기 때문에 case 1이 실행된것을 볼 수 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Condition/7.png){: width="100%" }

또한 switch 괄호 안에는 저런식으로 하나의 식을 넣어도 됩니다. 단, 그 값은 정수여야 겠죠?

**3\. 중첩(nested) 조건문**

중첩 조건문은 말그대로 조건문 안에 있는 조건문입니다. 바로 예시로 가보시죠

![Desktop View](/assets/img/Programming-Language/C/Condition/8.png){: width="100%" }

이런식으로 n의 값에 따라서 n%2 ==0 , 즉 n이 짝수냐에 따라 짝수라면 또 그 안에서 조건문 if 와 else를 볼 수 있습니다. 짝수이고 3의 배수면 if 안에 if를 실행시키고 짝수지만 3의 배수가 아니면 if 안에 else조건문을 실행시킵니다.

![Desktop View](/assets/img/Programming-Language/C/Condition/9.png){: width="100%" }

6을 입력했더니 if 가 실행되고 또 그 안에 if절이 실행되었음을 알 수 있습니다.

\*\*주의사항!!

1\. if(10<4);

이것 처럼 조건문 바로 뒤에 ;을 붙여 버리면 거기서 if 문장은 끝나게 됩니다. 따라서 그 뒤에 블럭은 if의 조건과 아무 상관없는 문장이므로 무조건 실행되기 때문에 주의 하셔야합니다!

2\. 비교연산자 ==와 대입연산자 =를 헷갈리지 않기!

수학공부를 할때는 부등호는 c언어에서도 같이 쓰지만 등호는 ==를 씁니다.

왜냐하면 =도 엄연히 대입연산자이기 때문! 역할이 다릅니다!

오늘은 조건문에 대해서 배웠습니다!
