---
title: "[EDA] 타이타닉 생존자 분석"
date: 2023-11-27 00:00:00 +0900
categories: [Bigdata-AI, EDA]
tags: [titanic, eda]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 타이타닉 데이터셋으로 **생존자 분석 EDA**를 진행한다. 결측치 확인·대치(특히 나이의 그룹별 평균 대치)부터, 좌석 등급·성별·연령대에 따른 생존률을 시각화로 파헤친다.

> **EDA 목표** — 단순히 결측치를 채우는 게 아니라, **원래 분포를 훼손하지 않으면서** 데이터를 보완하고, 어떤 요인이 생존에 영향을 주는지 찾는 것이다.

---

## 1. 결측치 확인

```python
plt.figure(figsize=(8,6))
msno.matrix(df)
plt.show()
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/1.png)

`age`에 결측치가 꽤 있고 `fare`는 거의 없다. `fare`는 평균으로 바로 대치해도 되지만, **`age`는 그냥 평균을 넣으면 분포가 왜곡**된다.

---

## 2. 나이 결측치 — 전체 평균의 문제

```python
df['age'].fillna(df['age'].mean(), inplace=True)
# original vs tuned 분포 비교(kdeplot)
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/2.png)

> ⚠️ **전체 평균으로 대치하면 평균 지점만 뾰족하게 솟는다.** 결측치가 모두 한 값(평균)으로 채워져, 원래의 완만한 분포 형태가 무너진다. 대치 전후 분포를 항상 비교해야 하는 이유다.

---

## 3. 그룹별 평균 대치 (개선)

성별·좌석 등급으로 세분화해 **각 그룹의 평균**을 결측치에 넣는다.

```python
df['age'].fillna(
    df.groupby(["sex","pclass"])['age'].transform('mean'),
    inplace=True)
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/3.png)

> 💡 **`groupby` + `transform('mean')`**이 핵심이다. `transform`은 그룹 평균을 **원본과 같은 길이로** 반환해, 각 행의 결측치에 그 행이 속한 그룹의 평균을 정확히 매핑한다. 결과적으로 전체 분포 형태를 유지하면서 볼륨만 커진다.

---

## 4. 좌석 등급 & 성별 생존률

```python
sns.FacetGrid(df, col='pclass', row='sex').map(sns.histplot, 'survived')
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/4.png)

**결과**: 등급이 좋을수록 생존률↑. 특히 **등급이 낮을수록 남성의 생존률이 급격히 하락**한다.

---

## 5. 연령대별 생존률

에버랜드 입장권 기준(?)으로 5개 연령대로 나눈다.

```python
def age_make(x):
    if x <= 3.0:   return "baby"
    elif x <= 13.: return "kid"
    elif x <= 18.: return "teenage"
    elif x <= 60.: return "adult"
    else:          return "old"

df['cat.age'] = df['age'].apply(age_make)
sns.barplot(data=df, x="cat.age", y="survived", hue="sex",
            order=['baby','kid','teenage','adult','old'])
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/5.png)

| 관찰 | 내용 |
|------|------|
| **여성** | 연령대가 **높아질수록 생존률↑** |
| **남성** | 연령대가 높아질수록 생존률↓ |

등급(pclass)으로 나눠보면 `kid`는 1·2등급이 전부 생존했다.

![Desktop View](/assets/img/Bigdata-AI/EDA/Titanic/6.png)

> 💡 **표본 크기를 항상 의식**하자. baby·old를 등급으로 또 3분할하니 각 그룹 표본이 너무 적어 신뢰하기 어렵다. 세분화는 유용하지만, 나눌수록 그룹당 데이터가 줄어드는 트레이드오프가 있다.

---

## 📝 정리

```
타이타닉 EDA
├─ 결측치   msno로 확인(age 많음)
├─ 대치     전체 평균(왜곡) → 그룹별 평균(groupby+transform)
├─ 분석     등급↑·여성 생존률↑, 낮은 등급 남성 급락
└─ 주의     세분화 시 표본 부족 유의
```

| 개념 | 한 줄 정의 |
|------|------|
| **EDA** | 데이터를 다각도로 탐색 |
| **transform('mean')** | 그룹 평균을 원본 길이로 |
| **분포 비교** | 대치 전후 kdeplot |

타이타닉 EDA의 핵심 교훈은 **결측치 대치는 분포를 지키며 해야 한다**는 것이다. 그룹별 평균 대치로 형태를 유지했고, 등급·성별·연령의 상호작용에서 "여성·고등급 우선 생존"이라는 뚜렷한 패턴을 확인했다.
