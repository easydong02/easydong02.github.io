---
title: 넘파이(Numpy)
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---
이번 시간엔 Numpy를 알아보겠습니다~

파이썬으로 데이터나 통계분석을 할 때 가장 많이 쓰이는 것이 numpy,pandas,matplotlib 등이 있죠! 그중에 하나입니다 ㅎㅎ

#### **Numpy**

-   넘파이에서는 배열을 효과적으로 관리하기 위한 ndarray 객체를 제공합니다. 이는 파이썬 기본 자료구조인 리스트 (튜플)을 업그레이드 해서 보다 향상된 기능을 제공합니다. 리스트를 상속받아 더 많은 기능을 구현해서 사용성을 높이는 것으로 추측해 볼 수 있죠?

```
import numpy as np

arr = [1, 2, 3, 4]
a = np.array(arr)
print( type (a) )

>>><class 'numpy.ndarray'>
```

이렇게 arr라는 리스트를 만들고 끝나는 것이 아니라 np.array() 메서드의 파라미터로 넣어주면 됩니다. type() 함수로 확인해볼 수 있겠죠? 이제 이 np로 할 수 있는 여러가지 메서드를 알아보고 기존의 파이썬 리스트와 어떤 점이 다른지 알아보겠습니다!

```
b = np.zeros(10)

b

>>>array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0.])



np.ones(10,dtype=int)

>>>array([1, 1, 1, 1, 1, 1, 1, 1, 1, 1])
```

zeros()는 파라미터의 숫자만큼의 공간을 가진 0을 가진 array로 만들어줍니다. 마찬가지로 ones()도 1로  채워서 array를 만들어줍니다.

```
np.ones((3,3), dtype=int)


>>>array([[1, 1, 1],
       [1, 1, 1],
       [1, 1, 1]])
```

ones() 파라미터에 튜플로 3,3을 넣어서 3행3열의 2차원 행렬로 만들어줍니다. 옵션으로 정수형으로 만들었습니다!

```
a = np.array([1,2])
b =  np.array([3,4])

c = np.vstack((a,b))
print (c)

>>>[[1 2]
 [3 4]]
```

vstack은 두 array를 수직방향으로 합쳐줍니다. 주의할 점은 합칠 때 길이가 같아야합니다. 반대로 hstack()은 수평으로 합쳐줍니다!

```
arr = [1, 2, 3, 4]
a= np.array(arr)
print(a)

>>>[1 2 3 4]


b = a.reshape([2,2])
print(b)

>>>[[1 2]
 [3 4]]
```

reshape()는 array를 원하는 행과 열의 사이즈로 재구성을 시켜줍니다. arr를 2,2의 행렬로 만들어봤습니다~

```
a = np.arange(2,10,2)
print(a)

>>>[2 4 6 8]
```

arange()는 리스트를 따로 만들필요 없이 바로 만들 수 있게 해줍니다~

```
a = np.array(range(12)).reshape(4,3)
print(a[0][1])

>>>1
```

이것은 0부터 11까지의 array를 4행 3열의 행렬로 만들고 인덱싱을 해서 첫행 두번째 열의 원소를 가져왔습니다!

```
a[:2 ,:]


>>>array([[0, 1, 2],
       [3, 4, 5]])
```

위에서 만든 a에서 콤마( , ) 는 행과 열을 나눠줍니다. 앞에선 처음부터 2행까지, 뒤에는 모든 열을, 즉 2차원 행렬에서 2행만 추출하겠네요~

이번에는 간단하게 matplotlib 맛보기로 시각화를 해볼게요~

```
x = np.linspace(0,1,100)
y = x **2

import matplotlib.pyplot as plt

plt.plot(x,y)
```

x에 linspace()로 0부터1까지 100으로 촘촘히 나눠서 y= x\*\*2 함수를 시각화했습니다.

![Desktop View](/assets/img/Programming-Language/Python/Numpy/1.png)

잘 나오네요 ㅎㅎ

```
import numpy as np

# x + y = 4
# 2x + 4y = 14 를 행렬식으로 표현

a = np.array( [
    [1, 1],
    [2, 4]
])

b = np.array([4, 14])
```

이번에는 행렬로 2차 방정식을 푸는 방법을 파이썬으로 구현하겠습니다. x, y 계수만 모은 행렬을 역행렬로 변환한 뒤 상수값이랑 곱하면 값이 나옵니다.(사실 이부분은 수학이라.. ㅎㅎ 다 아시죠..?)

```
inv_a = np.linalg.inv(a)

inv_a @ b  # @는 행렬곱셈 연산

>>>array([1., 3.])
```

이렇게 x와 y를 쉽게 구해봤습니다 ㅎㅎ 재밌네요

이번시간엔 여기까지 하겠습니다!