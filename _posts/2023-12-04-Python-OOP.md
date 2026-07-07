---
title: "[Python] OOP 딕셔너리 datetime"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

본격적으로 **객체지향(OOP)**에 들어간다. 상속·오버라이딩, 그리고 **리스트 내포·zip·datetime** 같은 실용 문법까지 정리한다.

---

## 1. 예외로 반복문 탈출하기

```python
classrooms = {'1반': [162,175,198,137,145,199], '2반': [165,177,157,160,199]}

try:
    for n in classrooms.keys():
        for a in classrooms[n]:       # 이중 for로 원소까지 접근
            if a > 190:
                print('{}에 190넘는 사람이 있습니다.'.format(n))
                raise StopIteration   # 예외로 반복 탈출
except StopIteration:
    print('190이 넘는 학생을 찾았습니다.')
```

> 💡 딕셔너리로 접근해도 원소까지 닿으려면 **이중 for**가 필요하다. 중첩 반복을 빠져나올 때 `break`를 두 번 쓰는 대신, **`raise StopIteration`**으로 한 번에 except로 넘어가면 유용하다.

---

## 2. 상속과 오버라이딩

부모 `Animal`을 `Human`, `Cat`이 상속한다.

```python
class Animal():
    def __init__(self, name):
        self.name = name
    def walk(self): print('걷는다')
    def eat(self):  print('먹는다')
    def greet(self): print('{}이/가 인사한다'.format(self.name))
```

```python
class Human(Animal):
    def __init__(self, name, hand):
        super().__init__(name)   # 부모 생성자 호출 (name 전달)
        self.hand = hand         # 자식은 hand만 처리
    def wave(self): print('{}을 흔든다'.format(self.hand))
    def greet(self):             # 오버라이딩 (자식이 우선)
        self.wave()              # self = 자기 자신
        super().greet()          # super() = 부모

person1 = Human('동희', '오른손')
person1.greet()
```

![Desktop View](/assets/img/Programming-Language/Python/OOP/1.png)

| 키워드 | 의미 |
|------|------|
| `super().__init__(name)` | 부모 생성자 호출 |
| `self` | 자기 자신 |
| `super()` | 부모 |

> 💡 같은 이름의 메소드를 재정의하면 **자식이 우선**된다. `Human.greet()`은 자신의 `wave()`와 부모의 `greet()`를 함께 호출한다.

```python
class Cat(Animal):
    def walk(self): print("우아하게 걷는다")   # walk만 재정의
```

---

## 3. 리스트 내포 · enumerate · zip

**리스트 내포(comprehension)** — 한 줄로 리스트 생성:

```python
print([n*n for n in range(1, 11) if n % 2 == 0])   # 짝수의 제곱
```

**enumerate** — 인덱스와 값을 함께:

```python
students = ['김개똥', '김말똥', '김소똥']
for num, name in enumerate(students):
    print('{}번 {} 학생'.format(num + 1, name))

# 딕셔너리 내포로도
students_dict = {"{}번".format(n+1): name for n, name in enumerate(students)}
```

**zip** — 여러 리스트를 짝지어 묶기:

```python
students = ['김개똥', '김말똥', '김소똥']
scores = [90, 98, 97]
for x, y in zip(students, scores):
    print(x, y)          # 김개똥 90 ...

for x in zip(students, scores):
    print(x)             # ('김개똥', 90) ... 튜플로
```

![Desktop View](/assets/img/Programming-Language/Python/OOP/4.png)

> 💡 `zip`은 복수의 리스트를 딕셔너리처럼 원소끼리 엮는다. `print(x, y)`면 둘씩, `print(x)`면 하나의 튜플로 출력된다.

---

## 4. datetime

```python
import datetime

print(datetime.datetime.now())   # 연,월,일,시,분,초,마이크로초
```

![Desktop View](/assets/img/Programming-Language/Python/OOP/6.png)

```python
stime = datetime.datetime.now()
stime.replace(hour=2)            # 특정 부분만 변경

# 날짜 뺄셈 → 남은 시간
christmas = datetime.datetime(2021,12,25,11,30,30) - datetime.datetime.now()
print(christmas)                 # 크리스마스까지 남은 시간
```

![Desktop View](/assets/img/Programming-Language/Python/OOP/7.png)

---

## 📝 정리

```
OOP · 실용 문법
├─ 상속       class 자식(부모): + super() + 오버라이딩(자식 우선)
├─ 리스트 내포  [식 for x in ... if 조건]
├─ enumerate  인덱스+값 동시 순회
├─ zip        여러 리스트를 짝지어 묶기
└─ datetime   now(), replace(), 날짜 뺄셈으로 기간 계산
```

| 개념 | 한 줄 정의 |
|------|------|
| **super()** | 부모 클래스 참조 |
| **리스트 내포** | 한 줄로 리스트 생성 |
| **zip** | 여러 리스트를 원소끼리 묶기 |
| **datetime** | 날짜·시간 처리 |

파이썬 OOP는 `super()`와 오버라이딩(자식 우선)만 잡으면 쉽다. 여기에 리스트 내포·zip·enumerate 같은 파이썬다운 간결한 문법을 곁들이면 코드가 훨씬 우아해진다.
