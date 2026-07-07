---
title: "[Python] 객체 클래스 생성자 모듈 예외처리"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 클래스에 이어 **생성자·상속·클래스 변수**, 자바의 import 같은 **모듈**, 그리고 **예외 처리**를 다룬다.

> **객체와 인스턴스**: `a = Cookies()`에서 `a`는 **객체**이자 **Cookies의 인스턴스**다. "a는 인스턴스다"보다 **"a는 Cookies의 인스턴스다"**가 더 정확한 표현 — 어떤 클래스로 만든 객체인지 관계를 드러내기 때문이다.

---

## 1. 생성자와 클래스 (계산기 예제)

파이썬 생성자는 **더블 언더바 `__init__`**으로 정의한다.

```python
class FourCal():
    def __init__(self, first, second):   # 생성자
        self.first = first
        self.second = second
    def setData(self, first, second):    # 값 수정
        self.first = first
        self.second = second
    def add(self): return self.first + self.second
    def mul(self): return self.first * self.second
    def div(self): return self.first / self.second
    def sub(self): return self.first - self.second
```

```python
a = FourCal(4, 2)   # 생성 시 first=4, second=2
b = FourCal(3, 1)
```

![Desktop View](/assets/img/Programming-Language/Python/Object/1.png)

> 💡 파이썬도 자바처럼 **생성자에 기본 매개변수를 강제**할 수 있다. `setData()`로 수정도 가능하지만, 두 값은 반드시 필요하므로 **생성 시점에 `self.first`, `self.second`를 설정**하도록 했다.

---

## 2. 상속

클래스명 뒤 **소괄호에 부모 클래스**를 넣으면 상속된다. 파이썬은 자바보다 상속이 간편하다.

```python
class MoreFourCal(FourCal):     # FourCal 상속
    def pow(self):
        result = self.first ** self.second   # 제곱 기능 추가
        return result
```

> 💡 부모(`FourCal`)의 속성·기능을 그대로 쓰면서 자식에서 **기능(제곱)을 추가**했다. 또, **클래스 변수**는 자바의 `static`처럼 인스턴스가 아니라 **클래스 이름으로 호출**한다.

---

## 3. 모듈

> **모듈**: 함수·변수·클래스를 모아 놓은 파이썬 파일. 다른 프로그램에서 불러와 재사용한다. 자바처럼 **`import`문**을 쓴다. (남이 만든 것도, 내가 만든 것도 사용 가능)

---

## 4. 예외 처리

자바의 `try~catch`와 유사하지만 **`except`**를 쓴다.

```python
try:
    a = [1, 2]
    print(a[1])
    4 / 0
except ZeroDivisionError as e:   # 에러 타입 + alias
    print('0으로 나누었습니다.')
except IndexError as e:
    print("인덱스에러")
finally:
    print('종료합니다.')          # 항상 실행
```

| 구문 | 역할 |
|------|------|
| `try` | 오류 가능성 있는 코드 |
| `except 에러 as e` | 특정 예외 처리 |
| `finally` | 항상 실행 |

> ⚠️ `try` 안에 여러 오류가 있으면 **가장 처음 만난 오류**에서 except로 넘어가고 finally로 종료된다. 따라서 코드 배치를 효율적으로 해야 한다.

### 사용자 정의 예외

```python
class MyError(Exception):   # Exception 상속
    pass                    # 내용 없음 (틀만)

def sayNick(nick):
    if nick == '바보':
        raise MyError()     # 예외 발생시키기
    print(nick)

try:
    sayNick('천사')
    sayNick('바보')
except MyError:
    print('허용되지 않는 별명입니다')
```

> 💡 모든 예외의 부모인 **`Exception`을 상속**해 커스텀 예외를 만든다. `pass`는 "틀은 갖추되 내용은 없음"을 뜻한다. **`raise`**로 원하는 시점에 예외를 발생시킬 수 있다.

---

## 📝 정리

```
객체·모듈·예외
├─ 생성자   __init__(self, ...) (기본값 강제 가능)
├─ 상속     class 자식(부모): + 기능 추가
├─ 모듈     import로 함수·클래스 재사용
├─ 예외     try / except 에러 as e / finally
└─ 커스텀   class MyError(Exception) + raise
```

| 개념 | 한 줄 정의 |
|------|------|
| **__init__** | 파이썬 생성자 |
| **인스턴스** | 특정 클래스로 만든 객체 |
| **except** | 예외를 잡아 처리(자바의 catch) |
| **raise** | 예외를 발생시킴 |

파이썬의 OOP는 자바와 개념이 같지만 **`__init__` 생성자, `import` 모듈, `try~except` 예외**라는 문법 차이만 익히면 된다. 특히 `raise`로 원하는 예외를 직접 발생시키는 방식이 유용하다.
