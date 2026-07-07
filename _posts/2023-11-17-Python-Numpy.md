---
title: "[Python] 넘파이(Numpy)"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

파이썬으로 데이터·통계 분석을 할 때 핵심 라이브러리는 **numpy, pandas, matplotlib**다. 이번 글에서는 그중 **Numpy**를 정리한다.

> **Numpy란?**
> 배열을 효과적으로 관리하는 **`ndarray` 객체**를 제공한다. 파이썬 기본 리스트를 업그레이드해 더 향상된 기능(벡터·행렬 연산)을 준다.

```python
import numpy as np

arr = [1, 2, 3, 4]
a = np.array(arr)
print(type(a))
# >>> <class 'numpy.ndarray'>
```

---

## 1. 배열 생성 메소드

| 메소드 | 역할 |
|------|------|
| `np.zeros(n)` | 0으로 채운 배열 |
| `np.ones(n)` | 1로 채운 배열 |
| `np.arange(s,e,step)` | 범위 배열 (리스트 불필요) |
| `np.array(...).reshape(r,c)` | 행·열 재구성 |

```python
np.zeros(10)                # [0. 0. ... 0.]
np.ones(10, dtype=int)      # [1 1 ... 1]

np.ones((3, 3), dtype=int)  # 3x3 2차원 행렬
# [[1 1 1]
#  [1 1 1]
#  [1 1 1]]

np.arange(2, 10, 2)         # [2 4 6 8]
```

**reshape** — 배열을 원하는 형태로 재구성한다.

```python
a = np.array([1, 2, 3, 4])
a.reshape([2, 2])
# [[1 2]
#  [3 4]]
```

---

## 2. 배열 합치기 (stack)

| 메소드 | 방향 |
|------|------|
| `np.vstack((a,b))` | 수직(세로) |
| `np.hstack((a,b))` | 수평(가로) |

```python
a = np.array([1, 2])
b = np.array([3, 4])
np.vstack((a, b))
# [[1 2]
#  [3 4]]
```

> ⚠️ 합칠 때 배열의 **길이가 같아야** 한다.

---

## 3. 인덱싱·슬라이싱

```python
a = np.array(range(12)).reshape(4, 3)   # 0~11을 4x3으로
a[0][1]     # 첫 행 두 번째 열 → 1

a[:2, :]    # 콤마로 행·열 구분: 2행까지, 모든 열
# [[0 1 2]
#  [3 4 5]]
```

> 💡 2차원 배열에서 **콤마(`,`)가 행과 열을 구분**한다. `a[:2, :]`는 "0~1행, 모든 열"을 의미한다.

---

## 4. 활용 — 시각화 & 행렬로 방정식 풀기

**함수 시각화** (matplotlib 맛보기):

```python
x = np.linspace(0, 1, 100)   # 0~1을 100등분
y = x ** 2

import matplotlib.pyplot as plt
plt.plot(x, y)
```

![Desktop View](/assets/img/Programming-Language/Python/Numpy/1.png)

**연립방정식을 역행렬로 풀기** — `x + y = 4`, `2x + 4y = 14`:

```python
a = np.array([[1, 1], [2, 4]])   # 계수 행렬
b = np.array([4, 14])            # 상수

inv_a = np.linalg.inv(a)         # 역행렬
inv_a @ b                        # @ = 행렬곱
# >>> array([1., 3.])   → x=1, y=3
```

> 💡 계수만 모은 행렬을 **역행렬**로 바꿔 상수 벡터와 **행렬곱(`@`)**하면 해가 나온다. 넘파이가 선형대수를 얼마나 간결하게 다루는지 보여주는 예다.

---

## 📝 정리

```
Numpy
├─ 생성    zeros/ones/arange/reshape
├─ 합치기  vstack(수직) / hstack(수평) — 길이 같아야
├─ 인덱싱  a[행, 열], 콤마로 구분
└─ 선형대수  linalg.inv(역행렬), @ (행렬곱)
```

| 개념 | 한 줄 정의 |
|------|------|
| **ndarray** | Numpy의 향상된 배열 객체 |
| **reshape** | 배열을 행·열로 재구성 |
| **@ (matmul)** | 행렬 곱 연산 |
| **linalg.inv** | 역행렬 |

Numpy는 "리스트를 벡터·행렬로 다루는" 데이터 분석의 기반이다. `reshape`·인덱싱·행렬곱만 익혀도 pandas·시각화로 자연스럽게 이어진다.
