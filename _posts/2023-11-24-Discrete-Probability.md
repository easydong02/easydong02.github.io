---
title: "[이산수학] 데이터 정리와 확률"
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath, probability]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **데이터 정리(대푯값·분산·시각화)**와 **확률(순열·조합·조건부확률·분포)**을 인공지능 관점에서 정리한다.

---

## 1. 대푯값 — 평균과 중앙값

| 대푯값 | 정의 | 특징 |
|------|------|------|
| **평균(mean)** | 모두 더해 개수로 나눔 | 가장 많이 쓰이나 **이상치에 민감** |
| **중앙값(median)** | 크기순 정렬 후 가운데 값 | 이상치에 강함 |

![Desktop View](/assets/img/Math/Discrete-Math/Probability/1.png)

---

## 2. 분산 (Variance)

> **분산**: 평균으로부터 **퍼짐의 정도**를 숫자로 표현.

```python
np.random.seed(0)
scores1 = np.random.randint(30, 100, 10)
scores2 = np.random.randint(50, 90, 10)   # scores1과 평균은 같음

print(scores1.var())   # 분산은 다를 수 있음
print(scores2.var())
```

![Desktop View](/assets/img/Math/Discrete-Math/Probability/3.png)

> 💡 **평균이 같아도 분산은 다를 수 있다.** 분산은 "평균으로부터의 편차를 한 변으로 하는 사각형의 평균 넓이"로도 볼 수 있다.

---

## 3. 데이터 시각화

| 그래프 | 용도 |
|------|------|
| **히스토그램** | 데이터를 계급으로 나눠 빈도를 막대로 (`bins`가 클수록 촘촘) |
| **상자그래프(Boxplot)** | 사분위수 + 펜스로 **이상치**를 구분 |

```python
# Boxplot의 핵심 계산
Q1 = np.percentile(D, 25)
Q3 = np.percentile(D, 75)
IQR = Q3 - Q1                    # 사분위 범위
upper_fence = Q3 + 1.5 * IQR     # 이상치 경계
lower_fence = Q1 - 1.5 * IQR
```

![Desktop View](/assets/img/Math/Discrete-Math/Probability/6.png)
![Desktop View](/assets/img/Math/Discrete-Math/Probability/7.png)

> 💡 Boxplot은 `Q1 - 1.5·IQR` ~ `Q3 + 1.5·IQR`를 벗어나는 값을 **이상치**로 표시한다.

---

## 4. 공분산과 상관계수

| 지표 | 의미 |
|------|------|
| **공분산(covariance)** | 두 데이터가 함께 퍼지는 정도 |
| **상관계수(correlation)** | 단위와 무관한 상관성 지표 (−1 ~ 1) |

**부호의 의미:**

| 값 | 관계 |
|:---:|------|
| 양수 | 양의 직선 관계 (X↑ → Y↑) |
| 음수 | 음의 직선 관계 (X↑ → Y↓) |
| 0 근처 | 직선 관계 없음 |

![Desktop View](/assets/img/Math/Discrete-Math/Probability/8.png)

> ⚠️ **공분산이 크다고 관계가 더 강한 건 아니다.** 데이터 수치(예: 몸무게를 g으로 표기)가 크면 공분산도 커지기 때문. 그래서 **단위 무관한 상관계수**를 쓴다.

> 💡 상관계수는 **두 편차 벡터 사이 각의 코사인 값**이라, 어떤 수를 넣어도 **−1 ~ 1**이 된다. (시그모이드가 0~1인 것과 비슷)

---

## 5. 순열과 조합

| 개념 | 정의 | (a,b)와 (b,a) |
|------|------|:---:|
| **순열** | n개 중 m개를 **일렬로 배열** | **다름** (순서 O) |
| **조합** | 순열에서 **순서 무시** | **같음** (순서 X) |

![Desktop View](/assets/img/Math/Discrete-Math/Probability/11.png)

> 💡 조합은 순열에서 순서가 같은 중복을 제거한 것이다.

---

## 6. 확률 분포

### 조건부 확률

> **조건부 확률** `P(B|A)`: 사건 A가 일어났을 때 사건 B가 일어날 확률. A를 조건으로 한다.

![Desktop View](/assets/img/Math/Discrete-Math/Probability/13.png)

### 주요 분포

| 분포 | 정의 | 예 |
|------|------|------|
| **베르누이(Bernoulli)** | 0 또는 1을 갖는 이진 확률변수 | 동전 던지기 |
| **멀티누이(Multinoulli)** | 이진 변수의 일반화(벡터 변수) | 주사위 던지기 |
| **결합확률분포(joint)** | 두 변수의 동시 확률 | |

![Desktop View](/assets/img/Math/Discrete-Math/Probability/14.png)
![Desktop View](/assets/img/Math/Discrete-Math/Probability/16.png)

> 💡 멀티누이 분포의 제약: **각 변수의 모든 확률을 더하면 1**이다.

---

## 📝 정리

```
데이터 정리 & 확률
├─ 대푯값   평균(이상치 민감) / 중앙값(강함)
├─ 분산     퍼짐 정도 (평균 같아도 다를 수 있음)
├─ 시각화   히스토그램 / Boxplot(이상치)
├─ 관계     공분산 / 상관계수(-1~1, 단위 무관)
├─ 경우의 수  순열(순서O) / 조합(순서X)
└─ 확률     조건부확률 / 베르누이·멀티누이·결합
```

| 개념 | 한 줄 정의 |
|------|------|
| **분산** | 평균으로부터 퍼짐 정도 |
| **상관계수** | 단위 무관한 상관성 (−1~1) |
| **순열 vs 조합** | 순서 있음 vs 없음 |
| **베르누이** | 이진 확률변수 분포 |

데이터 정리와 확률은 **머신러닝의 통계적 기반**이다. 특히 상관계수가 코사인 값이라 −1~1로 정규화된다는 점, 공분산은 단위에 좌우된다는 점은 데이터 분석의 핵심 통찰이다.
