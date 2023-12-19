---
title: "[C] 구조체"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

어느덧 C언어의 마지막 포스팅 시간이 왔네요.. 시간이 후딱갑니다 ㅎㅎ

일단 마지막 포스팅이라고 생각하지만 제가 또 올리고 싶은 것이 있다면 올리겠습니다. 현재는 C언어에 관한 문제를 제가 풀어보고 해석하는 방향으로 포스팅을 할 생각은 있습니다.

이번에는 구조체에 대해서 포스팅을 해보려고 합니다.

**구조체 (struct)**

여러개의 데이터들 (멤버) 로 구성된 사용자 정의 자료형입니다. 한 구조체에 여러가지 타입의 변수를 저장할 수 있습니다. 예를 들어,

학생의 데이터

\-학번 : char\[\]

\-이름 : char\[\]

\-나이 : unsigned char (0 ~ 255)

\-학년 : unsigned char

\-성별 : 'M', 'F' char

\-키 : 173.4 float

... 등등 이런 식으로 우리가 정한 구조체에 이것들을 선언하고 사용할 수 있다는 것이죠.

그렇다면 한번 구조체를 정의해보겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/1.png){: width="100%" }

대표사진 삭제

사진 설명을 입력하세요.

point라는 구조체를 만들었습니다. 위치는 main()함수 밖에 함수처럼 위치합니다. 멤버로는 int타입의 x,y 가 있네요.

이것들을 멤버 변수(member variable)이라고 합니다!

자그럼, 이것을 main()함수에서 사용해보도록 합시다.

![Desktop View](/assets/img/Programming-Language/C/Structure/2.png){: width="100%" }

먼저 p1이란 변수를 선언했습니다. 그렇다면 이 변수의 자료타입(type)은 무엇일까요? 네 맞습니다. point라는 이름의 구조체타입의 변수입니다. 그리고 이 변수에는 멤버변수가 2개지요? 그래서 대입할때는 멤버연산자(.)을 사용합니다. 따라서 p1.x =100;은 p1변수의 멤버변수 x에 100의 값을 대입한다는 것이지요. 마찬가지로 y도 이렇게 대입해준 뒤, printf문으로 출력해보겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/3.png){: width="100%" }

잘 들어가 있는 것을 확인할 수 있습니다. 재밌네요!

구조체에는 여러가지 타입이 있을 수 있습니다. 다른 구조체를 정의해보겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/4.png){: width="100%" }

person이란 구조체를 만들었습니다. 여기에는 char,int,double 타입의 멤버변수들이 있네요. 사용은 위와 마찬가지로 응용할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/5.png){: width="100%" }

person 구조체 타입의 p2변수를 main()함수에 선언했습니다. age와 weight에는 값을 직접 대입했는데요, 구조체에서 문자열을 입력받을 때에는 이렇게 합니다. 그리고 strcpy를 사용할 경우 소스파일 제일 위에 #include <string.h>를 써주도록 합시다!

![Desktop View](/assets/img/Programming-Language/C/Structure/6.png){: width="100%" }

잘 출력이 되었습니다!!

![Desktop View](/assets/img/Programming-Language/C/Structure/7.png){: width="100%" }

이렇게 구조체 변수선언과 동시에 배열의 형태로 초기화가 가능합니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/8.png){: width="100%" }

또는 이런 식으로 person 구조체타입의 p4변수를 0으로 초기화 할 수 있습니다. 모든 변수가 0으로 초기화 되는데요,그러면 char타입은 어떻게 되냐구요? NULL(0)문자로 초기화 되어서 출력하면 아무것도 없이 나옵니다~ 배열과 비슷한 부분이네요.

**typedef**에 대해서 알아보죠

이 문장은 사용자가 원하는 타입명으로 바꾸어서 사용할 수 있게 해줍니다. 간단히 말해서 main()함수 위에서 정의된 구조체를 본문에서 사용하려면 struct person p1; 이런식으로 적어야하는데 물론 얼마 걸리진 않겠지만 그 길이가 길어진다면 일일이 다 쓰기 복잡해지겠죠? 그래서 이것을 씁니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/9.png){: width="100%" }

아까 우리가 정의했던 person구조체입니다. 그밑에 typedef struct person PS; 라는 문장이 보이시나요?

이 문장은 앞으로 struct person을 PS로 쓸수 있게 만든 겁니다.

따라서 본문에 PS p1; 처럼 바로 person타입의 변수 p1을 선언할 수 있죠.

![Desktop View](/assets/img/Programming-Language/C/Structure/10.png){: width="100%" }

이런식으로 말이죠. 간편하죠? 또 다른 방법도 있습니다. 구조체 정의와 동시에 하는거죠.

![Desktop View](/assets/img/Programming-Language/C/Structure/11.png){: width="100%" }

보이시나요? \_person이라는 구조체를 정의할 때부터 typedef 와 구조체 블럭이 끝난 후 Person; 을 넣어서 앞으로 struct \_person 을 Person으로 쓸 수 있게 장치를 한 것입니다.

**구조체의 Size**

구조체의 크기는 어떻게 될까요? 바로 위의 Person을 봅시다. char형 문자가 20개 들어갈 공간이니 20byte, int는 4byte, double은 8byte니까 다 합쳐서 32byte가 나올까요?

printf("sizeof(Person) = %d\\n", sizeof(Person)); 문장으로 알아봅시다.

![Desktop View](/assets/img/Programming-Language/C/Structure/12.png){: width="100%" }

보이시는 것처럼 32byte 가 일단 출력이 되었습니다! 자세한 것은 추후에 또 배우면 말씀드릴게요!

**구조체에 대한 포인터**

구조체에도 포인터방식이 적용될 수 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/13.png){: width="100%" }

이렇게 Person 타입의 구조체에 대해서 맨 밑에보시면 포인터변수 ptr을 선언하고 그 주소값을 p3변수의 주소값을 넣었습니다. 포인터랑 같은 개념이죠.

이 포인터를 응용하기 전에 자주 쓰일거같은 말을 지정해 전역변수로 만들었습니다.

char \*fmt = "이름:%s, 나이:%d, 몸무게:%.1lf\\n";

![Desktop View](/assets/img/Programming-Language/C/Structure/14.png){: width="100%" }

이것처럼 말이죠. 그래서 맨 밑에 printf문 3개를 만들었습니다. 어떤 뜻일까요? 첫번째는 이름에 p3.name멤버변수와 age 와 weight를 출력하게 했습니다. 그 밑에는 포인터의 방식으로 넣었습니다. 멤버연산자(.)가 참조연산자(\*)보다 우선순위 이기 때문에 \*ptr에 괄호를 추가해 먼저 연산을 하도록 하였습니다. 출력은 첫번째 printf문과 같이 나오겠죠? 그 다음이 문제입니다 ->? 기호가 뭐죠? 두번째 printf문처럼 포인터를 사용해 출력을 원하는 경우, 저렇게 매번 복잡하게 적어야하기 때문에 C언어에서 간단하게 사용하기 위해 만든 기호입니다. (\*ptr).name을 그냥 ptr->name으로 더 간단하게 표현만 하는거죠 작동은 똑같습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/15.png){: width="100%" }

똑같이 Person 타입의 변수 p3의 내용을 그대로 나타내고 있습니다.

이를 이용해서 구조체 매개변수를 구조체의 주소(포인터)로 받는 함수를 설정할 수 도 있습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/16.png){: width="100%" }

void 함수 printfPerson 를 정의하고 이 함수의 매개변수는 Person타입의 포인터를 설정했습니다.

사용은 본문에서

printPerson(ptr); (ptr은 이미 Person \*ptr=&p3;로 설정해놨습니다.) 아니면

printPerson(&p3); 로 직접 할 수 있겠죠?

**구조체와 배열**

구조체를 배열의 형식으로도 사용 가능합니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/17.png){: width="100%" }

Person 타입의 배열 people을 만들었습니다. 다만 단순한 배열과는 다르게 배열 원소에 또 다른 멤버변수가 들어가 있다는 점입니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/18.png){: width="100%" }

이런식으로 사용을 한번 해보겠습니다.

![Desktop View](/assets/img/Programming-Language/C/Structure/19.png){: width="100%" }

ㅋㅋ 잘나오네요!

이번 시간에는 구조체에 대해서 길게 써봤습니다~~ 긴 여정이었네요 보람도 있고 아주 좋아요!
