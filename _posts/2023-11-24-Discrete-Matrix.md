---
title: "[이산수학] 행렬"
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath, matrix]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **행렬(Matrix)**을 정리한다. 인공지능의 연산이 대부분 **행렬 연산**으로 이루어지므로 매우 중요하다.

> **행렬**: 사각 괄호로 둘러싸인 숫자들의 배열.

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/1.png)

---

## 1. 행렬의 곱셈

> 덧셈은 요소끼리 더하면 되지만, **곱셈**은 규칙이 다르다. **행과 열의 인덱스를 각각 곱해 더한 값**이 결과의 한 원소가 된다.

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/2.png)

```python
import numpy as np

# 덧셈: 요소끼리
A = np.matrix([[1,2],[3,4]])
B = np.matrix([[10,20],[30,40]])
A + B

# 곱셈: np.dot 또는 @
A = np.array([[1,2,3], [4,5,6]])
B = np.array([[2,1], [1,2], [1,1]])
print(np.dot(A, B))   # 행렬곱
print(A @ B)          # @ (행렬곱 연산자)
# A * B 는 에러 (요소곱 시도)

# matrix 타입이면 * 로도 행렬곱
A_ = np.matrix(A); B_ = np.matrix(B)
A_ * B_
```

| 연산 | 방법 |
|------|------|
| 덧셈 | `A + B` (요소끼리) |
| 행렬곱 | `np.dot(A,B)` 또는 `A @ B` |

> ⚠️ `np.array`에서 `A * B`는 **요소곱**(또는 에러)이다. 행렬곱은 `@`나 `np.dot`을 써야 한다.

---

## 2. 여러 가지 행렬

| 종류 | 정의 |
|------|------|
| **전치(transpose)** | 행과 열을 뒤바꾼 행렬 |
| **단위 행렬** | 대각 성분이 모두 1인 정사각 행렬 |
| **대각 행렬** | 대각 성분만 0이 아닌 정사각 행렬 |

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/3.png)
![Desktop View](/assets/img/Math/Discrete-Math/Matrix/4.png)
![Desktop View](/assets/img/Math/Discrete-Math/Matrix/5.png)

---

## 3. 데이터 테이블로서의 행렬

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/6.png)

> 💡 행렬은 **데이터 테이블**을 표현할 때도 쓰인다. 한 개체(샘플)를 **행**으로, 그 개체의 특징(feature)들을 **열**로 표현한다. 머신러닝의 데이터셋이 바로 이 형태다.

```
        특징1  특징2  특징3
개체1 [  ...    ...   ...  ]
개체2 [  ...    ...   ...  ]
```

---

## 📝 정리

```
행렬
├─ 곱셈    행×열 인덱스 곱의 합 (@, np.dot)
├─ 전치    행↔열 교환
├─ 단위행렬  대각 1 (정사각)
├─ 대각행렬  대각만 값
└─ 데이터  행=개체, 열=특징 (ML 데이터셋)
```

| 개념 | 한 줄 정의 |
|------|------|
| **행렬곱** | 행×열 곱의 합 (`@`) |
| **전치** | 행과 열을 바꿈 |
| **단위/대각 행렬** | 대각 1 / 대각만 값 |

행렬은 인공지능 연산의 언어다. 특히 **행렬곱(`@`)**과 **데이터를 행(개체)·열(특징)로 표현**하는 관점은, 머신러닝의 모든 계산이 결국 행렬 연산이라는 사실로 이어진다.
