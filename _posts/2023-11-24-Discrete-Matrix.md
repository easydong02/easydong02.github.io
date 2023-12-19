---
title: "[이산수학] 행렬"
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath, matrix]
render_with_liquid: false
future: true
---

이번 시간에는 행렬을 알아보겠습니다. 인공지능의 연산이 행렬연산으로 이루어지므로 유용하게 쓰입니다!

#### **행렬?**

사각 괄호로 둘러 쌓인 숫자들의 배열

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/1.png)

이런식으로 나타내죠!

행렬의 연산은 다 아실테지만 곱셈은 중요하니 다뤄볼게요!

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/2.png)

말 그대로 행과 열의 인덱스를 각각 곱해서 더한값을 하나의 원소로 결과가 나옵니다.

```
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd

# 행렬과 행렬의 덧셈은 요소끼리 덧셈
A = np.matrix([[1,2],[3,4]])
B = np.matrix([[10,20],[30,40]])

A+B


# 행렬과 행렬의 곱셈
A = np.array([[1,2,3], [4,5,6]]) 
B = np.array([[2,1], [1,2], [1,1]])
print(np.dot(A,B)) #벡터의 곱하기

# 그냥 곱하면 에러가 남 
# A*B
print(A@B) #파이썬에서 지원하는 행렬곱 연산자
########################################
# 그냥 곱하고 싶으면?
A_ = np.matrix(A)
B_ = np.matrix(B)
A_*B_
```

#### **행렬의 전치**

행렬의 전치는 주어진 행렬의 행과 열을 뒤바꾼 구조입니다. 

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/3.png)

이렇게 말이죠!

단위 행렬

대각 성분이 모두 1인 정사각 행렬

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/4.png)

대각행렬

대각 성분만 0이 아닌 성분을 가진 정사각 행렬

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/5.png)

![Desktop View](/assets/img/Math/Discrete-Math/Matrix/6.png)

행렬은 이런식으로 데이터 테이블을 표현할 때 쓰기도 합니다. 한 개체의 데이터를 행으로, 특징들을 열 데이터로 표현하였습니다.

이번 시간엔 여기까지 하겠습니다!