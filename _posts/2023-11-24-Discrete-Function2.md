---
title: "[이산수학] 함수2"
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **여러 종류의 함수**(다항·지수·시그모이드·로그)와 **스칼라·벡터**를 다루며, 이들이 **인공지능에 어떻게 활용**되는지 살펴본다.

> **함수란?** 입력 변수가 어떤 연산을 거쳐 출력 변수로 나오는 구조. 첫 집합의 한 원소를 두 번째 집합의 **오직 한 원소**에 대응시키는 관계.

![Desktop View](/assets/img/Math/Discrete-Math/Function2/1.png)

---

## 1. 여러 함수의 종류

| 종류 | 정의 | 예 |
|------|------|------|
| **단항식** | 항 하나 | `2`, `x`, `x²`, `x/3` |
| **다항식** | 단항식의 합 | `3x² + 2` |
| **다항함수** | 다항식으로 된 함수 | `f(x) = x² + 2x + 3` |
| **지수함수** | 0보다 큰 a의 지수가 변수 | `aˣ` |

![Desktop View](/assets/img/Math/Discrete-Math/Function2/3.png)

---

## 2. 시그모이드 함수 (AI 활용)

> **로지스틱 시그모이드 함수**는 신경망에서 **뉴런의 활성을 결정하는 활성함수**로 쓰인다.

![Desktop View](/assets/img/Math/Discrete-Math/Function2/4.png)

> 💡 `z`에 **어떤 실수를 넣어도 0~1 사이** 값이 나온다. 그래서 출력을 **확률로 해석**할 수 있어, 분류 문제에 유용하다.

---

## 3. 로그함수 (AI 활용)

> **로그함수**는 지수함수를 `y=x`에 대칭시킨 함수다.

![Desktop View](/assets/img/Math/Discrete-Math/Function2/6.png)

**로그의 활용 — 오차율 계산:**

> 💡 `(예측값 - 정답)²`로 오차를 표현하면 문제가 생긴다. **10↔15**와 **1000↔1005**는 둘 다 `25`로 같은 오차가 되지만, 실제 의미는 다르다 (10→15는 50% 차이, 1000→1005는 미미). **로그를 붙이면** 상대적 차이를 더 정확히 반영할 수 있다.

![Desktop View](/assets/img/Math/Discrete-Math/Function2/7.png)

> 💡 또한 로그는 **중첩된 미분**이 필요할 때 계산을 간단하게 만들어준다.

---

## 4. 스칼라와 벡터

| 개념 | 정의 |
|------|------|
| **스칼라** | 크기만 있는 양 (한 번에 하나의 값) |
| **벡터** | 크기 + 방향으로 표현되는 양 (속도, 힘 / 숫자 여러 개) |

**입력과 출력 형태에 따른 함수 분류:**

![Desktop View](/assets/img/Math/Discrete-Math/Function2/8.png)

**다변수 스칼라 함수 시각화** (numpy + matplotlib):

```python
import numpy as np
import matplotlib.pyplot as plt

x1 = np.linspace(-2, 2, 20)
x2 = np.linspace(-1, 3, 20)
X1, X2 = np.meshgrid(x1, x2)
Z = 50*(X2 - X1**2)**2 + (2 - X1)**2   # 다변수 → 스칼라

ax = plt.axes(projection='3d')
ax.plot_surface(X1, X2, Z, cmap=plt.cm.binary, edgecolor="k")
plt.show()
```

![Desktop View](/assets/img/Math/Discrete-Math/Function2/9.png)

> 💡 변수가 2개면 (상수 c 고정 시) **등고선**, 3개면 **등고면**을 얻는다. 변수 3개 + 상수까지는 **4차원**이라 시각화할 수 없다.

---

## 📝 정리

```
함수2 (AI 활용)
├─ 종류    다항/지수/로그 함수
├─ 시그모이드  출력을 0~1 확률로 (활성함수)
├─ 로그    오차율 표현 개선 + 미분 간소화
└─ 스칼라/벡터  크기만 / 크기+방향
```

| 개념 | 한 줄 정의 |
|------|------|
| **시그모이드** | 값을 0~1로 압축(활성함수) |
| **로그** | 상대 오차 표현·미분 간소화 |
| **스칼라 vs 벡터** | 크기만 vs 크기+방향 |

이 함수들은 단순한 수학 개념을 넘어 **인공지능의 핵심 부품**이다. 시그모이드는 활성함수로, 로그는 손실함수 계산으로 쓰인다. 수학이 어떻게 AI로 연결되는지 보이기 시작하는 지점이다.
