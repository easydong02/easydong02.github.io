---
title: "[Python] 함수 사용자입력 반복문 클래스"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **반복문(for/range/내포)**, **함수(가변인자·다중반환·global·lambda)**, **사용자 입력**, **파일 입출력**, 그리고 **클래스**까지 폭넓게 리뷰한다.

---

## 1. 반복문

**커피 자판기** — while + 조건문:

```python
coffee = 10
while True:
    money = int(input("돈을 넣어 주세요 : "))
    if   money == 300: print("커피를 드립니다."); coffee -= 1
    elif money > 300:  print("거스름 돈은 %d원입니다." % (money-300)); coffee -= 1
    else:              print("금액이 부족합니다.")
    if coffee == 0:    print("매진되었습니다"); break
```

**for문** — 파이썬의 직관성을 잘 보여준다.

```python
list = [1, 2, 3, 4, 5]
for i in list:          # 원소를 하나씩
    print(i)

a = [(1,2), (3,4), (5,6)]
for x in a:             # 튜플 세트로
    print(x)            # (1,2), (3,4), (5,6)
for x, y in a:          # 튜플을 각 값으로 분해
    print(x + y)        # 3, 7, 11
```

**range** — `range(start, stop, step)` (start 생략 시 0, step 생략 시 1):

```python
for i in range(1, 11, 2): ...   # 1,3,5,7,9
for i in range(11, 1, -2): ...  # 11,9,7,5,3 (감소)

# 구구단
for i in range(2, 10):
    for j in range(1, 10):
        print(i * j, end=" ")
```

**리스트 내포** — 여러 줄을 한 줄로:

```python
a = [1, 2, 3, 4]
result = [n*3 for n in a if n % 2 == 0]   # 짝수만 3배 → [6, 12]
```

> 💡 조건(`if`)이 먼저 검사되고, 통과한 `n`이 연산되어 리스트에 담긴다.

---

## 2. 함수

함수는 **매개변수 유무 × 리턴값 유무**로 4종류가 있다.

**가변 인자** — 개수를 모를 때 `*` 사용:

```python
def add_many(*args):    # 여러 개 받기
    result = 0
    for i in args:
        result += i
    return result
add_many(1, 2, 3, 4, 5)   # 15
```

**다중 반환** — 튜플로:

```python
def add_mul(a, b):
    return a + b, a * b       # (7, 12) 튜플로 반환
result = add_mul(3, 4)        # (7, 12)
result1, result2 = add_mul(3, 4)   # 언패킹 → 7, 12
```

> 💡 리턴값은 언제나 **한 개**다. 식이 2개면 **튜플 하나**로 묶여 나온다. `return`을 만나면 함수는 즉시 종료되므로, 그 아래의 두 번째 `return`은 의미 없다.

**변수 범위와 global:**

```python
a = 1
def vartest(a):    # 매개변수 a는 함수 안에서만
    a = a + 1
vartest(a); print(a)   # 1 (원본 안 바뀜)

a = 1
def vartest2():
    global a           # 전역 a를 사용
    a = a + 1
    return a
```

**lambda** — 함수를 한 줄로:

```python
add = lambda a, b: a + b
add(3, 4)   # 7
```

---

## 3. 사용자 입력

> `input()`은 **문자열(str)만** 반환한다. 숫자로 쓰려면 `int(input())`.

```python
def num(a):
    if a % 2 == 1: print("홀수입니다.")
    else:          print('짝수입니다')
num(int(input()))

# choice로 로직 분기
def add_mul(choice, *args):
    if choice == "add":
        result = 0
        for i in args: result += i
    elif choice == "mul":
        result = 1
        for i in args: result *= i
    return result
print(add_mul('mul', 1, 2, 3, 5))   # 30
```

---

## 4. 파일 입출력

자바보다 훨씬 간단하다.

```python
f = open("파일이름.txt", 'w')   # 생성
f.close()                        # 항상 닫기
```

| 모드 | 역할 |
|:---:|------|
| `'w'` | 쓰기(새로) |
| `'r'` | 읽기 |
| `'a'` | 추가(append) |

```python
# 쓰기
f = open("새파일.txt", 'w')
for i in range(1, 11):
    f.write("%d번째 줄입니다.\n" % i)
f.close()

# 읽기 (한 줄씩)
f = open("새파일.txt", 'r')
while True:
    line = f.readline()
    if not line: break
    print(line)
f.close()

# read() → 전체를 하나의 문자열로
lines = open("새파일.txt", 'r').read()
```

**with** — 자동으로 닫아준다:

```python
with open("새파일.txt", 'w') as f:
    f.write("No pain No gain")   # close 자동
```

---

## 5. 클래스 맛보기

```python
class Calculator:
    def __init__(self):   # 생성자
        self.result = 0
    def add(self, num):
        self.result += num
        return self.result

cal1 = Calculator()
print(cal1.add(3))   # 3
print(cal1.add(3))   # 6 (result 누적)
```

> 💡 `self.result`가 객체의 상태로 남아 있어, `add`를 호출할 때마다 값이 **누적**된다.

---

## 📝 정리

```
함수·입력·파일·클래스
├─ for/range  원소 순회, range(s,e,step), 리스트 내포
├─ 함수       *args(가변) / 다중반환(튜플) / global / lambda
├─ 입력       input()은 str → int() 변환
├─ 파일       open('w'/'r'/'a') + close, with는 자동
└─ 클래스     __init__ + self로 상태 유지
```

| 개념 | 한 줄 정의 |
|------|------|
| **\*args** | 개수 모를 때 가변 인자 |
| **lambda** | 한 줄짜리 익명 함수 |
| **with open** | 파일을 자동으로 닫음 |
| **self.변수** | 객체의 상태 유지 |

파이썬의 함수·파일 처리는 자바보다 훨씬 간결하다. 특히 `*args`, `lambda`, `with open`은 파이썬다운 표현력을 잘 보여준다.
