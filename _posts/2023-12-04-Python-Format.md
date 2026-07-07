---
title: "[Python] 패킹 포맷"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

돌아온 파이썬 시간! 이번 글에서는 **문자열 포맷(`.format`)**, **딕셔너리·튜플**, **패킹/언패킹**, **enumerate** 등 실전 문법을 정리한다.

---

## 1. 문자열 포맷 (.format)

C의 `%d`와 비슷한 로직이다. **중괄호 `{}`**에 값을 순서대로 채운다.

```python
n = 20
greeting = '안녕하세요'
place = '뉴월드'
welcome = '환영합니다.'

print('{}번 손님 {}, {}에 오신것을 {}!'.format(n, greeting, place, welcome))
# >>> 20번 손님 안녕하세요, 뉴월드에 오신것을 환영합니다.!
```

---

## 2. 리스트 조작

```python
list2 = [1, 2, 3]
list3 = list2 + [16]     # + 로 추가
del list3[-1]            # -1 = 마지막 인덱스 삭제
```

**enumerate** — 리스트에 **번호(인덱스)를 붙여준다.**

```python
bts = ['RM', '진', '슈가', '제이홉', '지민', '뷔', '정국']
for i in enumerate(bts):
    print('{}'.format(i))   # (0, 'RM'), (1, '진'), ...
```

![Desktop View](/assets/img/Programming-Language/Python/Format/1.png)

---

## 3. 웹페이지 가져오기 (urllib)

```python
import urllib.request

def getweb(url):
    response = urllib.request.urlopen(url)   # url 열기
    data = response.read()                   # 읽기
    return data.decode('utf-8')              # 디코딩

url = input('웹페이지 주소를 입력하세요')
print(getweb(url))   # 예: http://www.naver.com → HTML 출력
```

![Desktop View](/assets/img/Programming-Language/Python/Format/2.png)

---

## 4. 딕셔너리

> **딕셔너리**: **key-value** 쌍으로 구성. key로 value를 조회한다. **인덱싱·슬라이싱은 불가.**

**가위바위보** 예제 — key가 value를 이기도록 설계.

```python
wintable = {"가위": '보', '보': '바위', '바위': '가위'}

def rsp(mine, yours):
    if mine == yours:             return "비겼다"
    elif wintable[mine] == yours: return "내가 이겼다"
    elif mine == wintable[yours]: return "내가 졌다"

print(rsp(input(), "바위"))
```

**수정** — 리스트처럼 `del`, `pop()` 사용 가능.

```python
dict = {'one': 1, 'two': 2, 'three': 3}
dict['two'] = 22        # 수정
dict['four'] = 4        # 추가
del(dict['one'])        # 삭제
dict.pop('three')       # 삭제
```

**items()** — key와 value를 쌍으로 순회.

```python
names = {'Jane': 32, 'Tod': 35, 'Paul': 62}
for i, j in names.items():
    print('{}의 나이는 {}입니다.'.format(i, j))

# * 로 언패킹하면 더 간단히
for i in names.items():
    print('{}의 나이는 {}입니다.'.format(*i))
```

---

## 5. 튜플과 패킹/언패킹

> **튜플**: 수정할 수 없는 자료구조. 수정하려면 리스트로 바꿨다가 되돌린다.

```python
t1 = (1, 2, 3, 4, 5)
l1 = list(t1)     # 튜플 → 리스트
l1[0] = 6         # 수정
t1 = tuple(l1)    # 리스트 → 튜플
```

**패킹/언패킹:**

```python
c = (3, 4)
d, e = c          # 언패킹 (d=3, e=4)
f = d, e          # 패킹 (f=(3,4))
```

> 💡 여러 변수에 한 번에 나눠 담는 것이 **언패킹**, 다시 묶는 것이 **패킹**이다. `*` 기호는 복수의 변수를 다룰 때 쓴다.

```python
list = [1, 2, 3, 4, 5]
for a in enumerate(list):
    print('{}번 값: {}'.format(*a))   # * 로 (인덱스, 값) 언패킹
```

![Desktop View](/assets/img/Programming-Language/Python/Format/3.png)

---

## 📝 정리

```
포맷 · 자료구조 · 패킹
├─ .format   중괄호 {}에 값 채우기
├─ enumerate 리스트에 인덱스 붙이기
├─ 딕셔너리   key-value, items()로 쌍 순회
├─ 튜플       수정 불가 (list로 변환 후 수정)
└─ 패킹/언패킹  d,e=c (언패킹) / f=d,e (패킹), * 활용
```

| 개념 | 한 줄 정의 |
|------|------|
| **.format** | 중괄호에 값을 채우는 문자열 포맷 |
| **딕셔너리** | key로 value를 조회 |
| **튜플** | 수정 불가한 순서 자료구조 |
| **패킹/언패킹** | 값 묶기 / 나눠 담기 |

파이썬은 "우리가 생각하는 것을 대부분 문법으로 구현해둔" 느낌이다. 특히 패킹/언패킹과 `*` 활용은 파이썬다운 간결함을 보여준다.
