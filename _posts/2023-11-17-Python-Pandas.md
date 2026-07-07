---
title: "[Python] 판다스(Pandas) - Series"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 Pandas의 **Series**를 다룬다.

> **Pandas란?**
> 데이터 분석을 위한 **고수준의 자료구조와 분석 도구**를 제공하는 라이브러리. 여기서 '고수준'은 데이터를 쉽게 제어·시각화하는 메소드를 뜻한다. 데이터 분석의 필수 모듈이다.

> **Series**: 인덱스가 붙은 1차원 데이터 구조.

---

## 1. Series 생성

```python
data = np.array(["가", "나", "다"])
s = Series(data)
# 0    가
# 1    나
# 2    다   (기본 인덱스 0,1,2)
```

**인덱스를 직접 지정**할 수 있다.

```python
data = [1000, 2000, 3000]
index = ['메로나', '구구콘', '하겐다즈']
s = Series(data=data, index=index)
# 메로나     1000
# 구구콘     2000
# 하겐다즈    3000
```

Series의 속성을 확인할 수 있다.

| 속성 | 반환 |
|------|------|
| `s.index` | 인덱스 |
| `s.values` | 값 |
| `s.dtype` | 자료형 |

---

## 2. 인덱싱 — loc vs iloc

| 방식 | 기준 | 슬라이싱 |
|------|------|------|
| `s.loc['메로나']` | **내가 만든 인덱스** | 마지막 **포함** |
| `s.iloc[0]` | **기본(정수) 인덱스** | 마지막 제외 |

```python
s.loc['메로나']   # 1000 (라벨로)
s.iloc[0]        # 1000 (정수 위치로)
```

> ⚠️ `loc`로 슬라이싱하면 `iloc`와 달리 **마지막 요소도 포함**한다.

---

## 3. 인덱스가 다른 Series 연산

```python
s1 = Series([10, 20, 30], index=["가", "나", "다"])
s2 = Series([20, 30, 40])   # 인덱스 0,1,2

s1 + s2   # 전부 NaN!
```

> 💡 Series 연산은 **인덱스를 기준으로 정렬**해서 계산한다. 인덱스가 맞지 않으면 짝이 없어 **NaN**(float 타입)이 된다.

---

## 4. 조건(불리언) 인덱싱

```python
lge = Series([93000, 82400, 99100, 81000, 72300],
             index=["05/27", "05/28", "05/29", "05/30", "05/31"])

lge.index[lge.values < 85000]
# >>> Index(['05/28', '05/30', '05/31'])   → 종가 85000 미만 날짜
```

> 💡 `values < 85000`은 각 데이터에 조건을 대입해 **True/False Series**를 만든다. 이를 인덱싱에 넣으면 **True인 것만** 추출된다. (LG전자 종가에서 85000 미만인 날짜)

---

## 번외 — 비트코인·리플 상관관계 회귀

두 코인 가격의 관계를 **역행렬(최소제곱)**로 일차함수로 도출한다.

```python
btc_re = btc['close'].values[-1000:].reshape(-1, 1)
xrp_re = xrp['close'].values[-1000:].reshape(-1, 1)

o = np.ones((1000, 1), dtype=np.uint32)
btc_o = np.hstack((btc_re, o))
btc_o_t = np.linalg.pinv(btc_o)   # 유사 역행렬

result = btc_o_t @ xrp_re
a, b = result[0, 0], result[1, 0]   # 기울기, 절편

x = btc['close'].values[-1000:]
y = a * x + b                        # 회귀선
plt.plot(x, y, color="r")
```

![Desktop View](/assets/img/Programming-Language/Python/Pandas/1.png)

x축을 비트코인, y축을 리플로 두고 대응값을 일차함수로 표현해 회귀선을 그렸다.

---

## 📝 정리

```
Pandas Series
├─ 생성    Series(data, index=...)
├─ 속성    index / values / dtype
├─ 인덱싱  loc(라벨, 끝 포함) vs iloc(정수)
├─ 연산    인덱스 기준 정렬, 안 맞으면 NaN
└─ 조건    values<n → True/False → 필터링
```

| 개념 | 한 줄 정의 |
|------|------|
| **Series** | 인덱스가 붙은 1차원 데이터 |
| **loc / iloc** | 라벨 인덱싱 / 정수 인덱싱 |
| **불리언 인덱싱** | 조건으로 데이터 필터 |
| **NaN** | 인덱스 불일치 시 결측값 |

Series의 핵심은 **"인덱스가 붙은 데이터"**라는 점이다. 그래서 연산이 인덱스 기준으로 이뤄지고, 조건 인덱싱으로 원하는 데이터를 손쉽게 걸러낼 수 있다. 다음엔 2차원인 DataFrame을 다룬다.
