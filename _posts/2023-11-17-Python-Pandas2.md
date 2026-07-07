---
title: "[Python] 판다스(Pandas) - DataFrame"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

저번엔 1차원 Series를 봤다. 이번엔 한 차원 더 나아간 **DataFrame**을 다룬다.

> **DataFrame이란?**
> 행과 열로 구성된 **2차원 데이터(엑셀 데이터)**를 다루는 데 효과적인 자료구조. 여러 개의 Series가 모인 것으로 볼 수 있다.

![Desktop View](/assets/img/Programming-Language/Python/Pandas2/1.png)

---

## 1. DataFrame 생성

**딕셔너리로 생성** (키가 열이 됨):

```python
from pandas import DataFrame

딕셔너리 = {"x": [10, 40], "y": [20, 30]}
df = DataFrame(딕셔너리)
#     x   y
# 0  10  20
# 1  40  30
```

**2차원 리스트로 생성**:

```python
리스트 = [[10, 20], [30, 40]]
df = DataFrame(리스트)
#     0   1
# 0  10  20
# 1  30  40
```

**인덱스·컬럼 지정**:

```python
df.index = ["x", "y"]      # 행 라벨(왼쪽 세로)
df.columns = ["a", "b"]    # 열 라벨(위쪽 가로)
```

![Desktop View](/assets/img/Programming-Language/Python/Pandas2/2.png)

> 💡 DataFrame의 각 **컬럼이 하나의 Series**이고, 반대로 **row도 Series**가 될 수 있다. Series를 2차원으로 확장한 것이 DataFrame이다.

---

## 2. 조건(불리언) 인덱싱

```python
df[df["시가"] >= 90]["종가"]         # 시가 90 이상인 행의 종가

cond = df["시가"] >= 90               # True/False Series
df.loc[cond, ["시가", "종가"]]        # 조건에 맞는 행의 특정 열
```

> 💡 `df["시가"] >= 90`이 **True/False Series**를 만들고, 이를 `df.loc[]`에 넣으면 **True인 행만** 추출된다. 원하는 조건을 만족하는 부분 DataFrame을 얻는다.

---

## 3. groupby — 그룹별 분류

```python
df = DataFrame({
    'sex':    ['m', 'm', 'w', 'm', 'w'],
    'weight': [76, 88, 54, 70, 45],
    'height': [176, 190, 148, 177, 155]
})

df_ = df.groupby("sex")     # sex의 고유값으로 그룹핑
df_.get_group("m")          # 'm' 그룹만 추출
```

> 💡 `groupby()`는 한 컬럼의 **고유값(unique)들로 데이터를 묶는다.** 그룹별로 추출하거나 집계(평균·합계 등)에 활용한다.

---

## 📝 정리

```
Pandas DataFrame
├─ 생성    DataFrame(딕셔너리) 또는 2차원 리스트
├─ 라벨    df.index(행) / df.columns(열)
├─ 구조    컬럼·row 모두 Series
├─ 조건    cond = df["열"] >= n → df.loc[cond, [...]]
└─ 그룹    groupby("열").get_group(값)
```

| 개념 | 한 줄 정의 |
|------|------|
| **DataFrame** | 행·열 2차원 데이터 (엑셀형) |
| **loc 조건 인덱싱** | 조건을 만족하는 행 추출 |
| **groupby** | 고유값으로 그룹핑 |

DataFrame은 "Series를 모은 2차원 표"다. 조건 인덱싱과 groupby만 익히면, 엑셀로 하던 데이터 필터·집계를 코드로 훨씬 유연하게 처리할 수 있다.
