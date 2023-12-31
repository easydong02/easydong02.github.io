---
title: "[Python] 클래스"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이번 시간에는 클래스를 알아보겠습니다. OOP의 꽃이라고 할 수 있는 부분인데 저는 c와 java배우면서 어느정도 기본지식이 있던 상태에서 들어서 그런지 좀더 편하게 들을 수 있었습니다.ㅎㅎ; 그래도 쉬운 개념은 아닌 것 같습니다.

#### **클래스란?**

-   클래스는 함수와 같은 의도로 코드를 정리하기 위해 사용합니다. 함수 보다 한 발 더 나아가 관련된 함수와 변수를 하나로 묶어서 정리할 수 있습니다. 관련 있는 통계학/경제학/정치학을 사회과학이라는 보다 큰 카테고리로 묶는 것과 같습니다.

```
class 사회과학:
    def 통계학 (self):
        print("통계학")

    def 경제학 (self):
        print("경제학")

    def 정치학 (self):
        print("정치학")
```

이런식으로 함수들을 사회과학이라는 클래스로 한데 묶어 객체마다 각자의 개성으로 메서드를 사용할 수 있게 됩니다.

다른 함수들과 다른 점은 클래스 안에 있는 함수들은 첫번째 파라미터로 self를 담고 있는데 이것은 나중에 이 클래스를 객체화 시켰을 때 그 객체가 들어가게 됩니다. 호출할때 물론 생략하셔도 됩니다.

```
변수 = 사회과학()

변수.경제학()


>>>경제학
```

이렇게 변수라는 이름으로 사회과학 클래스를 객체화 시켜서 이 변수라는 객체로 메서드를 호출하여 사용한 모습입니다. 얼핏보면 더 복잡해진 것 같지만 사실 대규모 프로젝트에서 다형화된 클래스를 이용하기 위해선 필수적입니다!

![Desktop View](/assets/img/Programming-Language/Python/Class/1.png)

이런 식으로 사회과학 클래스의 객체인 변수1,2를 각각 만들면 메모리에 다른 공간에 할당됩니다.

```
class 수업시간:
    def 예제1(self):
        return "어려워요"

    def 예제2(self, 값):
        print("입력:", 값)
```

수업시간이라는 클래스인데요 두 메서드의 차이점이라면 첫번째는 호출했을 때 "어려워요"라는 문자열값이 반환된 것이고 두번째는 파라미터로 들어온 값을 print()함수로 출력을 한것입니다. 따라서 예제1만 호출하면 아무런 출력이 없고

print(예제1)처럼 해야 값이 출력되겠죠?

```
class 예의바른학생:
    def 첫인사(self,이름):
        print("안녕하세요")
        print("반갑습니다")
        print(f"내이름은 {이름}!")

    def 작별인사(self):
        print("안녕히가세요")
        print("다음에 뵙겠습니다")    
        
        

홍길동 = 예의바른학생()
#FM 예의바른학생.첫인사("홍길동","무지")

홍길동.첫인사("신동희")
홍길동.작별인사()

>>>안녕하세요
>>>반갑습니다
>>>내이름은 신동희!
>>>안녕히가세요
>>>다음에 뵙겠습니다
```

위에서 클래스를 선언하고 홍길동이라는 객체를 만들어서 각각의 메서드를 활용하였습니다. 다른 방법으로는 객체를 첫부분에 쓰지않고 클래스.메서드(객체,"무지")처럼 self파라미터에 객체를 넣어서 활용해줄 수도 있습니다.

```
class 사람:
    def 이름입력 (self, 이름):
        self.이름 = 이름

    def 출력하기 (self):
        print(self.이름)

첫째 = 사람()
첫째.이름입력("신동희")
print(첫째.이름)

>>>신동희
```

이것은 이름입력()메서드에 문자열값을 넣어서 그 객체(첫째)가 가진 변수를 만들어줄 수도 있습니다. 첫째라는 객체의 이름이란 변수엔 "신동희" 값이 대입되어 있습니다.

이번 시간엔 여기까지 하겠습니다!!