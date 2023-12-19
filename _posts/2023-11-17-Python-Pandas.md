---
title: "[Python] 판다스(Pandas) - Series"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이번 시간엔 파이썬의 pandas에 있는 series기능을 이용해보겠습니다~

판다스(Pandas) 라이브러리

-   Pandas는 데이터 분석을 위한 고수준의 자료구조와 데이터 분석 도구를 제공합니다.

-   여기서 고수준이란 사용자가 쉽게 데이터를 제어하고 시각화 할 수 있는 메서드를 의미합니다.
-   판다스는 데이터 분석분야 에서 필수적으로 사용되는 중요한 모듈입니다.

```
data = np.array(["가", "나", "다"])
s = Series(data)
print(s)

>>>0    가
>>>1    나
>>>2    다
>>>dtype: object
```

data를 np.array로 선언한 다음에 series로 만들었습니다.

```
data = [1000, 2000, 3000]
index = ['메로나', '구구콘', '하겐다즈']
s = Series(data=data, index=index)
print(s)

>>>메로나     1000
>>>구구콘     2000
>>>하겐다즈    3000
>>>dtype: int64
```

series 옵션으로 data와 index에 우리가 만든 데이터를 넣어줄 수도 있습니다. 아까와 달리 0,1,2 인덱스가 아닌 아이스크림 이름으로 인덱스를 만들었습니다~

```
print(s.index)
print(s.values)
print(s.dtype)


>>>Index(['메로나', '구구콘', '하겐다즈'], dtype='object')
>>>[1000 2000 3000]
>>>int64
```

시리즈 객체의 변수로 우리가 만든 시리즈의 속성을 파악할 수도 있습니다.

```
data = [1000, 2000, 3000]
index = ['메로나', '구구콘', '하겐다즈']
s = Series(data, index=index)

print(s.loc['메로나']) # 만든 인덱스
print(s.iloc[  0   ])  # 디폴트 인덱스
# 인덱싱


>>>1000
>>>1000
```

인덱싱할 때 loc와 iloc가 있습니다. loc는 만든 인덱싱을 입력하여 추출하고 iloc는 리스트의 그 인덱싱과 비슷합니다. 주의할 점은 loc를 슬라이싱 할 때는 iloc와 달리 마지막 부분도 포함해서 추출합니다!

```
s1 = Series([10, 20, 30], index = ["가", "나", "다"])
s2 = Series([20, 30, 40])

s1+s2  # NaN이 나온 이유: index가 맞지 않아서.


>>>가   NaN
>>>나   NaN
>>>다   NaN
>>>0   NaN
>>>1   NaN
>>>2   NaN
>>>dtype: float64
```

인덱스가 다른 두 series를 +연산을 하면 각자 원래의 원소에 Nan값을 더해서 nan이 되고 결국 전체가 nan으로 출력됩니다. 참고로 nan은 float타입입니다!

```
lge = Series([93000, 82400, 99100, 81000, 72300], index = ["05/27", "05/28", "05/29", "05/30", "05/31"])

print(lge.index[ lge.values<85000])


>>>Index(['05/28', '05/30', '05/31'], dtype='object')
```

저렇게 원하는 인덱스에 조건문을 달면 전체 data를 조건에 대입하여 True와 False가 각각 대응하는 리스트로 만들어줍니다. 그것을 인덱싱하면 True인 값만 추출됩니다. 위는 lg전자의 종가를 data로 그 날짜를 index로 설정한 series입니다. 따라서 저 출력문은 종가가 85000미만인 날짜를 출력합니다~

번외) 비트코인과 리플의 가격 상관관계를 역행렬로 일반함수를 도출해보기!

```
import pybithumb
btc = pybithumb.get_candlestick("BTC")
xrp = pybithumb.get_candlestick("XRP")

plt.scatter(btc["close"][-1000:],xrp["close"][-1000:])

btc_re = btc['close'].values[-1000:].reshape(-1 ,1)
xrp_re = xrp['close'].values[-1000:].reshape( -1,1)

o = np.ones( (1000,1), dtype=np.uint32)
btc_o= np.hstack( (btc_re,o))
btc_o_t = np.linalg.pinv(btc_o)

result = btc_o_t @ xrp_re
a = result[0,0]
b = result[1,0]

plt.scatter(btc["close"][-1000:],xrp["close"][-1000:])

x = btc['close'].values[-1000:]
y = a*x +b  # 회귀

plt.plot(x,y,color="r")
```

![Desktop View](/assets/img/Programming-Language/Python/Pandas/1.png)

x축을 비트코인 y축을 리플로 잡아서 비트코인을 대입했을 때 대응되는 값을 일반함수로 표현하고 그래프로 그려봤습니다 ㅎㅎ

이번시간엔 여기까지 하겠습니다!