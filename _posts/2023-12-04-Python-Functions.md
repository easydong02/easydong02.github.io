---
title: "[Python] 여러가지 함수들"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 파이썬으로 배운 **여러 함수와 문자열 처리 기법**을 리뷰한다. 변수명 규칙, `print` 옵션, 슬라이싱 응용, 진법 변환, `is`/문자열 메소드까지 정리한다.

---

## 1. 변수명 규칙 & print 옵션

**변수명**: 첫 문자는 문자 또는 언더바. 단, **언더바 2개(`__`)로 시작하는 이름은 특별한 의미**(`__init__`, `__name__`, `__main__`)가 있으니 피한다. 키워드(`if`, `for` 등)도 피한다.

**print 옵션:**

| 옵션 | 역할 |
|------|------|
| `sep=문자열` | 값 사이의 구분자 (기본 빈칸) |
| `end=문자열` | 출력 후 마지막 문자열 (기본 줄바꿈) |

```python
side1 = float(input('한변의 길이'))
side2 = float(input('다른 한 변의 길이'))
hyp = ((side1**2) + (side2**2)) ** 0.5   # 피타고라스

print('빗변의 길이 : ', hyp, end=' ')     # 줄바꿈 대신 공백
print("입니다.")                          # 같은 줄에 이어짐
```

---

## 2. 함수 예제

**피보나치** — 다중 대입으로 간결하게:

```python
def fibo(n):
    a, b = 1, 0
    while a <= n:
        print(a, end=" ")
        a, b = a + b, a   # 동시 대입
fibo(5)   # 1 1 2 3 5
```

**업다운 게임** — 난수:

```python
from random import randint
n = randint(1, 500)   # 범위 내 무작위 정수
while True:
    ans = int(input("어떤 숫자일까요?"))
    if   ans > n: print("Down")
    elif ans < n: print("Up")
    else:         print("정답!"); break
```

---

## 3. 문자열 — 불변성과 슬라이싱

> 문자열은 **불변(immutable)**이다. 내용이 바뀌면 **새 주소값**을 부여받는다.

```python
mystr = "문자열은 불변"
print(id(mystr))        # 주소값
mystr = "숫자열은 불변"
print(id(mystr))        # 다른 주소값!
```

**슬라이싱 `[시작:끝:step]`:**

```python
a = "RoboCop"
print(a[1::2])          # ooo (인덱스 1부터 끝까지, 2칸씩)

def reverse_str(s):
    return s[::-1]      # step -1 → 거꾸로
reverse_str("12345")    # 54321
```

**불변 문자열 다루기** — 리스트 + `join()`:

```python
n = ord('A')            # 'A'의 아스키코드
s = []
for i in range(n, n + 26):
    s.append(chr(i))    # 리스트는 변경 가능 → 하나씩 추가
l = ''.join(s)          # 리스트 원소를 간격 없이 연결 → "ABC...Z"
```

> 💡 `a = a + "Bad"`처럼 이어붙이는 건 사실 매번 **새 주소**를 배정하는 것. 리스트에 담아 `join()`하면 효율적으로 문자열을 조립할 수 있다.

---

## 4. 진법 변환 & is 계열 메소드

```python
n = int('10001', 2)     # '10001'을 2진수로 인식 → 10진수 17
```

**`is` 계열** — 모두 **bool**을 반환한다.

| 메소드 | True 조건 |
|------|------|
| `isalnum()` | 글자+숫자로만 |
| `isalpha()` | 알파벳으로만 |
| `isdecimal()` | 10진수 숫자로만 |
| `islower()` / `isupper()` | 전부 소/대문자 |
| `isspace()` | 전부 공백 |
| `istitle()` | 첫 글자만 대문자 |

**startswith / endswith:**

```python
str = 'Sir. John Bennett. PhD'
str.endswith('PhD')     # True
str.startswith('Sir')   # True
```

---

## 5. split & strip

**split** — 문자열을 나눠 리스트로:

```python
'Moe Larry   Curly  Shemp'.split()      # 공백 무시하고 단어만
'Moe Larry   Curly'.split(' ')          # 공백 하나만 구분자로 (빈 요소 생김)
```

![Desktop View](/assets/img/Programming-Language/Python/Functions/1.png)

**strip** — 양 끝 공백 제거:

```python
'      Will Shakes      '.strip()       # 양쪽 공백 제거
# lstrip() 왼쪽만, rstrip() 오른쪽만
```

---

## 📝 정리

```
여러 함수 · 문자열
├─ print 옵션  sep(구분자) / end(끝문자)
├─ 다중 대입   a, b = a+b, a
├─ 슬라이싱     [::2](step), [::-1](역순)
├─ 불변 조립    리스트 append → ''.join()
├─ 진법        int('10001', 2) → 17
├─ is 계열     isalnum/isalpha/... (bool)
└─ split/strip 나누기 / 양끝 공백 제거
```

| 개념 | 한 줄 정의 |
|------|------|
| **문자열 불변** | 변경 시 새 주소 배정 |
| **[::-1]** | 문자열 뒤집기 |
| **join** | 리스트를 문자열로 연결 |
| **split/strip** | 나누기 / 공백 제거 |

파이썬의 문자열 처리는 **슬라이싱과 메소드**가 강력하다. 특히 `[::-1]`(역순), `join()`(조립), `split()`/`strip()`(가공)은 실전에서 끝없이 쓰이는 필수 기법이다.
