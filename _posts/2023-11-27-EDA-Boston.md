---
title: "[EDA] 보스턴 부동산 가격 분석"
date: 2023-11-27 00:00:00 +0900
categories: [Bigdata-AI, EDA]
tags: [bostonhousing, eda]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 보스턴 부동산 데이터셋으로 **집값(MEDV)에 영향을 주는 요인**을 찾는 EDA를 진행한다. 상관관계 히트맵으로 핵심 지표를 찾고, 상권/비상권·가격대별로 데이터를 나눠 숨은 패턴을 관찰한다.

> **EDA란?** 다양한 각도에서 데이터를 관찰·이해하는 과정. 이해도가 높아지면서 **숨은 의미와 잠재적 문제**를 발견하고, 이를 바탕으로 가설을 보완·수정한다.

---

## 1. 데이터 확인 & MEDV 분포

```python
df = pd.read_excel('BostonHousing.xls')
df.head()

sns.histplot(data=df, x="MEDV")   # 목표 변수 분포
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Boston/3.png)

20,000달러 부근에 몰려 있는데, **50,000달러 지점이 튀어** 조사가 필요하다.

```python
df.loc[df["MEDV"] >= 50]   # 50 이상은 전부 50.0
```

> ⚠️ **50 이상이 모두 50.0**이었다. 상한값을 잘라(censoring) 기록한 데이터로, 이런 인위적 절단은 분석을 왜곡할 수 있으니 반드시 짚고 넘어가야 한다.

---

## 2. 상관관계 히트맵

```python
df_corr = df.corr()
sns.heatmap(data=df_corr, annot=True, cmap="Blues_r")
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Boston/5.png)

| MEDV와 유의미한 상관(|r|≥0.6) | 의미 |
|------|------|
| **RM** | 방 개수(+) |
| **LSTAT** | 하위계층 밀집율(−) |

> 💡 `CAT. MEDV`는 MEDV가 30 이상이면 1인 **2차 파생 지표**라 MEDV와 상관이 높은 게 당연하다. 이런 파생 변수는 상관 분석에서 걸러 봐야 진짜 요인을 찾을 수 있다.

---

## 3. 상권 vs 비상권으로 나눠보기

부동산 가격은 **지역 성질에 따라 영향 요인이 다를 것**이라는 가설로, `INDUS`(상업 비율) 15를 기준으로 나눴다.

```python
sns.histplot(data=df, x="INDUS")   # 15 기준으로 두 군집
df_low = df.loc[df['INDUS'] < 15]    # 비상권
df_high = df.loc[df['INDUS'] >= 15]  # 상권
```

![Desktop View](/assets/img/Bigdata-AI/EDA/Boston/6.png)

| 지역 | 관찰 |
|------|------|
| **비상권** | 여전히 RM·LSTAT의 영향 |
| **상권** | **RM의 영향이 다르게** 나타남 |

![Desktop View](/assets/img/Bigdata-AI/EDA/Boston/8.png)

> 💡 데이터를 **하위 그룹으로 쪼개면 전체에서 안 보이던 차이**가 드러난다(심슨의 역설 회피). 다만 눈대중(15)보다는 **분위수(quantile)**로 나누는 게 더 객관적이다.

---

## 4. 가격대(CAT. MEDV)별 분석

```python
df_cat_1 = df.loc[df["CAT. MEDV"] == 1]   # 3만 달러 이상
df_cat_0 = df.loc[df["CAT. MEDV"] == 0]   # 3만 달러 미만
```

| 가격대 | 결과 |
|------|------|
| **고가(≥3만)** | 집값에 크게 영향 주는 지표 **없음** |
| **저가(<3만)** | 여러 지표가 영향 |

저가 구간에서 영향 큰 4개(CRIM·NOX·AGE·TAX)를 산점도로 살펴봤지만 뚜렷한 방향은 안 보였다.

![Desktop View](/assets/img/Bigdata-AI/EDA/Boston/11.png)

> 💡 **고가 주택은 가격 결정 요인이 복잡·개별적**이라 단순 지표로 설명되지 않는 반면, 저가 주택은 범죄율·세금 등 여러 환경 요인에 민감하다. 가격대를 나눴기에 발견한 인사이트다.

---

## 📝 정리

```
보스턴 부동산 EDA
├─ 목표     MEDV(집값) 영향 요인 찾기
├─ 핵심지표 RM(+)·LSTAT(-) (상관 0.6↑)
├─ 세분화   상권/비상권, 가격대별로 나눠 관찰
└─ 발견     저가는 환경요인 민감, 고가는 요인 불명확
```

| 개념 | 한 줄 정의 |
|------|------|
| **상관 히트맵** | 변수 간 관계 시각화 |
| **그룹 분할** | 하위 집단별 패턴 발견 |
| **파생 지표** | 상관 분석에서 제외 |

보스턴 EDA의 핵심은 **전체 상관관계에서 시작해, 데이터를 의미 있는 그룹으로 쪼개 숨은 패턴을 찾는 것**이다. 상권/가격대 분할로 "저가 주택은 환경에 민감, 고가는 요인이 복잡"이라는 인사이트를 얻었다.
