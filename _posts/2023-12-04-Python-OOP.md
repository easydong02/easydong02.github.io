---
title: "[Python] OOP 딕셔너리 datetime"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이제 객체지향 개념을 들어가기 시작했습니다. 동시에 점점 어려워지기도 하네요~ 그래도 어려운것을 배우면 문제를 보다 편하게 풀 수 있으니까 좋은 도구를 배운다는 마음으로 시작해봅시다~

```
classrooms ={'1반':[162,175,198,137,145,199],'2반':[165,177,157,160,199]}

try:
    for n in classrooms.keys():
        for a in classrooms[n]:
            if a>190:
                print('{}에 190넘는 사람이 있습니다.'.format(n))
                raise StopIteration
except StopIteration:
    print('190이 넘는 학생을 찾았습니다.')
```

딕셔너리로 2개의 반을 만들고 그 밸류값을 하나의 리스트로 만들었습니다. 그래서 리스트 중 하나의 원소가 190보다 크면 반복문을 멈추고 출력하는 코드를 만들었습니다! 여기서 주의할 점은 리스트로 접근했다고 그 원소에 접근한 것은 아니기 때문에 이중 for문으로 한번 더 접근했습니다.

그리고 중요한 것은 반복문을 빠져나올때 break를 써도 되지만 전 StopIteration예외를 raise로 발생시켜서 except문에 출력을 했습니다. 이렇게 하면 break를 2번쓰지않아도 아예 예외문으로 넘어가버리기 때문에 유용합니다!

**객체지향프로그래밍**

객체지향관련 부분은 자바든 파이썬이든지 어렵고 광범위하죠.. 하지만 알게되면 그만큼 편한것도 없습니다! 저는 먼저 모든 클래스를 상속해주는 Animal이라는 클래스를 만들고 human과 cat이 그것을 상속받아 여러가지 기능을 살펴볼까 합니다!

//Animal클래스

```
class Animal():
    def __init__(self,name):
        self.name=name
    def walk(self):
        print('걷는다')
    def eat(self):
        print('먹는다')
    def greet(self):
        print('{}이/가 인사한다'.format(self.name))
```

생성자를 만들고 매개변수에 name을 받도록 하였습니다! 그리고 greet()메서드엔 그 name을 바탕으로 하는 출력문을 작성하였습니다.

//Human클래스

```
class Human(Animal):
    def __init__(self,name,hand):
        super().__init__(name)  #부모의 생성자를 호출
        self.hand=hand          #자식은 hand에 관련된 부분만 처리
    def wave(self):
        print('{}을 흔든다'.format(self.hand))
    def greet(self):
        self.wave()     #self는 자기자신
        super().greet() #super()는 부모

person1 = Human('동희','오른손')


person1.greet()
```

Human 클래스에는 일단 Animal클래스를 상속받았구요 생성자를 보시죠, 부모클래스와 마찬가지로 name을 받는데 신기한 건 hand도 받았습니다. 그리고 name은 다시 super()를 통해서 부모생성자에 그 값을 전달했습니다. hand는 이 클래스에 대입을 했씁니다.

그래서 greet()함수를 보시면 부모랑 같은 이름의 메서드죠? 하지만 여기서 이렇게 같은 이름으로 재정의했을때는 먼저 우선순위가 된다는점! 그래서 그 메서드엔 wave()메서드를 호출하고 부모클래스의 greet()메서드를 호출합니다. 따라서 자식 클래스를 호출할 때 넘겨받은 name이 그대로 부모클래스에서 사용이 됩니다. 그럼 객체 person1을 만들고 거기에 '동희'와 '오른손'을 적었습니다 이때 greet()을 실행하면 어떻게 되는지 보시죠!

![Desktop View](/assets/img/Programming-Language/Python/OOP/1.png)

잘 출력이 되네요!!

//Cat클래스

```
class Cat(Animal):
    def walk(self):
        print("우아하게 걷는다")

cat = Cat()

cat.walk()
```

Cat클래스에는 walk()메서드를 재정의했습니다.

```
print([ n*n for n in range(1,11) if n%2==0])
```

리스트 내포를 이용해서 1부터 11미만까지 짝수를 제곱하여 출력하는 코드입니다

![Desktop View](/assets/img/Programming-Language/Python/OOP/2.png)

잘 나옵니다!

```
students =['김개똥','김말똥','김소똥','김똥똥','김벌똥','김새똥']

for num, name in enumerate(students):
    print('{}번 {} 학생'.format(num+1,name))
```

똥자돌림 학생 리스트를 만들었습니다. 이것을 딕셔너리의 조건제시법으로 출력했습니다, enumerate()로 딕셔너리처럼 만들어준 뒤 num,name을 차례로 출력합니다!

![Desktop View](/assets/img/Programming-Language/Python/OOP/3.png)

잘 나오네요 ㅎㅎ

```
students =['김개똥','김말똥','김소똥','김똥똥','김벌똥','김새똥']
students_dict={ "{}번".format(number+1):name for number,name in enumerate(students)}
print(students_dict)
```

이것은 비슷해보이지만 딕셔너리로 바꾸었습니다~

**zip()함수**

```
students =['김개똥','김말똥','김소똥','김똥똥','김벌똥','김새똥']
scores=[90,98,97,100,96,92,93]

for x,y in zip(students,scores):
    print(x,y)
```

zip()함수는 복수의 리스트를 인자로 받아서 딕셔너리처럼 각각의 원소를 엮어줍니다!

학생들과 점수 리스트를 만들어서 zip()으로 엮은 뒤 출력을 했습니다.

![Desktop View](/assets/img/Programming-Language/Python/OOP/4.png)

잘 출력되네요!

지금은 print(x,y)로 2개씩 출력했지만 그냥 print(x)를 한다면 어떻게 될까요?

```
for x in zip(students,scores):
    print(x)
```

![Desktop View](/assets/img/Programming-Language/Python/OOP/5.png)

이렇게 하나의 튜플로 쭉 출력합니다. 똑똑하네요! 이거로 튜플은 복수로도 하나로도 출력할 수 있다는 사실~

**Datetime**

```
import datetime

print(datetime.datetime.now())
```

파이썬에서는 datetime 라이브러리를 사용하여 날짜와 시간을 다룰 수 있다.

![Desktop View](/assets/img/Programming-Language/Python/OOP/6.png)

print()로 출력하면 저렇게 연,월.일,시,분,초,마이크로초로 나옵니다!

```
stime = datetime.datetime.now()

stime.replace(hour=2)
```

이번엔 현재시간을 stime의 객체에 넣어주고 replace()로 hour부분을 제가 원하는 값으로 변경했습니다!

```
christmas = datetime.datetime(2021,12,25,11,30,30,3000)-datetime.datetime.now()

print(christmas)
```

크리스마스 시간에서 현재 시간을 빼서 그것을 christmas객체에 넣었습니다! 이것을 출력하면?

![Desktop View](/assets/img/Programming-Language/Python/OOP/7.png)

현재로부터 크리스마스까지 남은 시간을 알려줍니다!