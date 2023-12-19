---
title: "[Python] 판다스(Pandas) - DataFrame"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

저번 시간엔 Series를 봤는데요 이번 시간엔 한 차원을 더 나아가 DataFrame을 보도록 하겠습니다.

판다스 데이터프레임

-   판다스 데이터프레임은 다음 그림과같이 행과 열로 구성된 2차원 데이터(엑셀 데이터)를 다루는데 효과적인 자료구조입니다

![Desktop View](/assets/img/Programming-Language/Python/Pandas2/1.png)

```
from pandas import DataFrame
import pandas as pd
import numpy as np

딕셔너리 = {
        "x" : [10,40],
        "y" : [20,30]
}
df = DataFrame(딕셔너리)
df

>>>	    x 	y
>>>0 	10 	20
>>>1 	40 	30
```

이렇게 딕셔너리를 만들어 준뒤 DataFrame()메서드에 파라미터로 넣어주어서 만듭니다. numpy를 시작으로 파이썬의 여러가지 수학 라이브러리들은 벡터연산을 기본으로 해야하기 때문에 열벡터로 나타낸 복수의 벡터로 나옵니다.

```
리스트 = [
    [10,20],
    [30,40]
]

df = DataFrame(리스트)
df

>>>  0 	  1
0 	10 	20
1 	30 	40
```

딕셔너리 뿐만 아니라 2차원 리스트를 넣으면 그 모양대로 이쁘게 데이터프레임으로 나오네요 ㅎㅎ

그리고 생성된 df에

```
df.index = ["x",'y']
df.columns=["a","b"]
```

이렇게 주면 index는 열방향으로 맨 왼쪽에 커스텀 인덱스가 들어가고 columns는 행방향으로 생깁니다.

![Desktop View](/assets/img/Programming-Language/Python/Pandas2/2.png)

이렇게요..

![Desktop View](/assets/img/Programming-Language/Python/Pandas2/3.png)

쉽게 이야기 하자면, 데이터 프레임의 각 컬럼이 하나의 series이거나 반대로 row가 series가 될 수 도 있습니다.

```
df[df["시가"] >=90]["종가"]
cond = df["시가"]>=90
df.loc[ cond, ["시가",'종가']]
```

이것은 조건을 만들어서 그 조건을 대응해보는 것인데요, df\['시가'\] >=을 하면 변수 cond엔 True와 False만 있는 Series로 반환해줍니다. 그리고 그것을 df.loc\[\]에 넣으면 True에 대응되는 원소만 출력이 되어서 결과적으로 우리가 원하는 조건을 만족하는 df를 볼 수 있게 됩니다.

```
df = DataFrame({
    'sex'    : ['m', 'm', 'w', 'm', 'w'],
    'weight' : [76, 88, 54, 70, 45],
    'height' : [176, 190, 148, 177, 155]        
})

df_ =df.groupby("sex")
df_.get_group("m")
```

이것은 groupby()메서드를 사용하여 한 series안의 unique값들을 모아주어 분류를 합니다. 그리고 그 그룹만 출력하거나 다른 용도로도 쓰일 수 있습니다~

이번 시간엔 여기까지 하겠습니다!