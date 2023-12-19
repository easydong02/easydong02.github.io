---
title: "[Python] 객체 클래스 생성자 모듈 예외처리"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이번 시간에는 저번에 배웠던 클래스에 이어서 조금 더 해보고 자바에서의 import같은 모듈, 그리고 중요한 예외처리를 알아보겠습니다!

클래스는 반복되는 생성과정을 쉽게 편하게 구성합니다. 붕어빵틀.. 이죠!

객체와 인스턴스.. 클래스로 만든 객체(인스턴스)... a = Cookies() a는 객체 Cookie라는 클래스 a는 Cookies의 인스턴스입니다!인스턴스라는 표현은 어떤 객체가 어떤 클래스로 만든 객체인지를 관계위주로 클래스로 설명할때 사용합니다. a는 인스턴스다.. 라는 표현보다는 a는 Cookies의 인스턴스다라는 표현이 더 정확하다고 할 수 있죠. 또는 a는 Cookies의 객체이다라던지!

그래서 이번엔 4가지 연산이 들어있는 계산기 클래스를 만들었습니다.

```
class FourCal():
  def __init__(self,first,second):
    self.first = first
    self.second = second

  def setData(self, first , second): #def 함수 -> 클래스 안에서는 메서드 self는 접근한 인스턴스
    self.first = first
    self.second = second
  def add(self):
    result = self.first + self.second
    return result
  def mul(self):
    result = self.first * self.second
    return result
  
  def div(self):

      result = self.first/self.second
      return result

    
  def sub(self):
     result = self.first - self.second
     return result
```

파이썬은 생성자 방식이 특이한데요 더블 언더바 (\_\_)를 사용해서 def로 정의를 합니다. 그리고 자바와 마찬가지로 파이썬에서도 생성자에 기본 매개변수를 강제할 수 있습니다. 물론 제가 이 클래스에 setData()라는 메서드를 만들어서 수정할 수 있도록 하였지만 일단 2개의 값을 분명히 필요하므로 바꿀 때는 바꾸더라도 인스턴스를 생성할때부터 self.first와 self.second를 인자를 통해 설정하도록 하였습니다. 그리고 그 밑에 add,mul,div,sub 메서드를 넣어서 각각 들어간 인자들을 어떤 연산을 이용할지 정할 수 있습니다.

```
a = FourCal(4,2)
b = FourCal(3,1)
```

인스턴스 a,b 를 각각 생성자 매개변수에 4,2와 3,1을 넣어준 뒤 클래스 안에 있는 메서드들을 호출해보았습니다.

![Desktop View](/assets/img/Programming-Language/Python/Object/1.png)

잘 나오네요 ^^

상속도 배웠습니다. 파이썬은 자바보다 상속이 더 편하게 할 수 있습니다. 자바와 같이 기존클래스를 변경하지 않고 기능을 추가하거나 기존 기능을 변경할 때 사용합니다.

```
class MoreFourCal(FourCal):
  def pow(self):
    result = self.first ** self.second   # 제곱을 계산하는 기능을 추가..
    return result
```

클래스를 하나 더 만들었습니다. 여기서 눈여겨 볼 것은 클래스명 뒤에 소괄호()에 아까만든 FourCal을 넣어주었다는 점, 따라서 상속이 발생합니다. 우리는 여기에 그치지 않고 부모클래스의 속성과 기능도 사용하고 자식에선 더 추가를 하고 싶군요! 그래서 제곱연산을 해주는 메서드도 만들었습니다.

재밌네요 ㅎㅎ

살짝만 다뤄보았지만 클래스변수라는 것도 배웠습니다. 마치 자바에서 static 변수를 사용할 때 인스턴스의 이름을 사용하는게 아니라 클래스이름으로 호출하는 것처럼 같으 맥락이라고 보시면 됩니다!

**모듈**

모듈은 함수, 변수, 클래스 등을 모아 놓은 파일입니다. 모듈은 다른 파이썬 프로그램 만들 때 불러와 사용할 수 있게끔 만든 파이썬 파일. 실제로 파이썬 프로그래밍을 할 때 많은 모듈을 사용하게 됩니다.다른 사람이 만든 모듈을 사용할 수도 있고 자신이 만든 모듈을 사용할 수도 있습니다. 문법은 자바와 같이 import문을 사용합니다!

**예외처리**

프로그래밍을 하면서 오류를 피해갈 수 없습니다. 논리적 오류일 수도 있고 외부의 결함이 있을 수도 있죠. 따라서 우리는 미연의 오류를 방지할 문장을 만들줄 알아야 합니다.

```
try:
  a=[1,2]
  print(a[1])
  4/0

except ZeroDivisionError as e:
  print('0으로 나누었습니다.')

except IndexError as e:
  print("인덱스에러")

finally:
  print('종료합니다.')
```

자바에서는 try~ catch로 하는데 파이썬은 try는 같지만 except라는 문장을 사용합니다. 그 뒤에는 에러의 이름과 그것을 alias로 우리가 원하는 이름을 지어준 뒤 우리의 입맛에 맞게끔 처리를 할 수 있습니다.그리고 finally는 아시다시피 마지막에 항상 실행되는 구문입니다. 주의할 점은 만약 try의 바디 안에 여러 오류 문장이 있다면 가장 처음 마주한 오류에서 except문으로 넘어가고 finally로 가서 코드가 끝납니다. 따라서 코드를 짤 때 하나하면 또 다른 하나하고.. 이런식으로 효율적인 배치가 필요합니다.

**예외만들기**

```
class MyError(Exception):
  pass


def sayNick(nick):
  if nick == '바보':
    raise MyError()

  print(nick)

try:
  sayNick('천사')
  sayNick('바보')
  
except MyError:
  print('허용되지 않는 별명입니다')
```

예외를 만들었습니다. MyError라는 클래스를 만들고 모든 예외의 공통부모인 Exception클래스를 상속받았습니다. 그리고 내용엔 pass로 변수로 따지자면 null과 같은 역할입니다. 클래스라는 형식은 갖춰졌지만 내용은 없는.. 그런 역할입니다. 따라서 함수 sayNick을 만들고 매개변수에 '바보'라는 문자열이 입력되었다면 raise명령어로 MyError)를 호출합니다. 따라서 '바보'를 넣으면 에러가 나겠죠? 그래서 try~except문으로 MyError가 난다면 어떻게 처리할지도 설계를 하였습니다~ 정말 재밌네요 ㅎㅎ

이번 시간에는 여기까지 하겠습니다!