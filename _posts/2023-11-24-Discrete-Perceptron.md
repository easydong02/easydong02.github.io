---
title: "[이산수학] 수 체계 기호 조건 퍼셉트론"
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath, perceptron]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **수 체계·데이터 인코딩·수학 기호·필요/충분조건**, 그리고 인공신경망의 시초인 **퍼셉트론**을 다룬다. 수학이 인공지능 이론과 결합되니 복잡하지만 그만큼 재미있다.

---

## 1. 수 체계

| 수 | 정의 |
|------|------|
| **자연수** | 셀 때·순서 매길 때 (1, 2, 3…) |
| **정수** | 자연수 + 0 + 음수 |
| **유리수** | 분수(정수/정수)로 나타낼 수 있는 수 |
| **무리수** | 비율(ratio)로 나타낼 수 없는 수 (e, π) |

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/1.png)

---

## 2. 데이터 인코딩 — 숫자로 표현하기

> AI 분석을 위해서는 모든 데이터를 **숫자로 표현**해야 한다. 문자로 된 범주형 데이터(주 손잡이 등)를 숫자로 변환한다.

```python
# 정수 변환
df['Age'] = df['Age'].astype('Float32').astype('Int16')

# OrdinalEncoder: 범주를 구분 숫자로
from sklearn.preprocessing import OrdinalEncoder
ord_enc = OrdinalEncoder()
ord_enc.fit(df[['Hand']])
ord_enc.transform(df[['Hand']])
```

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/2.png)

> ⚠️ Ordinal 인코딩은 "어떤 숫자가 더 크다"는 잘못된 순서 정보를 줄 수 있다. 그래서 **원핫 인코딩(One-Hot)**으로 **해당되는 곳에만 1**을 표시하기도 한다.

```python
from sklearn.preprocessing import OneHotEncoder
oh_enc = OneHotEncoder(sparse=False)
oh_enc.fit(df[["Hand"]])
oh_enc.transform(df[["Hand"]])
```

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/3.png)

| 인코딩 | 특징 |
|------|------|
| **Ordinal** | 범주를 정수로 (순서 정보 부여) |
| **One-Hot** | 해당 자리에만 1, 나머지 0 (순서 정보 없음) |

---

## 3. 수학 기호

| 기호 | 의미 |
|:---:|------|
| **∑ (시그마)** | 수열의 첫째 항부터 n항까지의 **합** |
| **∏ (프로덕트)** | 순서대로 모두 **곱함** |

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/5.png)

---

## 4. 필요조건과 충분조건

> 💡 갑자기 왜 필요/충분조건? 나중에 **손실 함수(loss function)를 미분해 로컬 미니멈을 찾을 때** 쓰인다. 도함수 `f'(x) = 0`이 최솟값의 **최소한의 조건(필요조건)**이 되는 것이다.

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/8.png)

---

## 5. 퍼셉트론 (Perceptron)

> **퍼셉트론**: 인공신경망의 한 종류. **1957년 프랑크 로젠블라트**가 고안했다. 논리 게이트(부울 대수의 회로 기본 단위)를 활용한다.

**동작 원리:**

```
입력 x, y  ──(가중치 곱)──►  합산 + 상수  ──►  시그모이드  ──►  분류
                                              (양수→1, 0 이하→0)
```

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/10.png)

> 💡 변수와 가중치를 곱해 더하고, 그 값을 시그모이드에 넣어 **양수면 1, 0 이하면 0**으로 분류한다. 예를 들어 **AND 로직**은 x, y **둘 다 1이어야** 합이 양수가 되도록 가중치·상수를 설정한다.

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/11.png)

### XOR 문제 — 다층 퍼셉트론

![Desktop View](/assets/img/Math/Discrete-Math/Number-Type/12.png)

> ⚠️ **XOR 논리**는 단층 퍼셉트론으로 구현할 수 없어, **제1차 인공지능의 겨울**을 불러왔다. 해결책은 **층을 하나 더 추가**하는 것이다. **NAND와 OR을 먼저 연산한 뒤, 그 결과를 다시 AND로 연결**하면 XOR이 나온다.

```
x, y ─┬─► NAND ─┐
      │         ├─► AND ─► XOR
      └─► OR ────┘
```

> 💡 이것이 바로 **다층 신경망**의 아이디어다. 단순한 층으로 못 풀던 문제를, 층을 쌓아 해결한다.

---

## 📝 정리

```
수 체계·퍼셉트론
├─ 수 체계    자연수⊂정수⊂유리수, 무리수(e,π)
├─ 인코딩     Ordinal / One-Hot (문자→숫자)
├─ 기호       ∑(합) · ∏(곱)
├─ 조건       필요조건 = 최소 조건 (f'(x)=0)
└─ 퍼셉트론    가중합 → 시그모이드 → 분류
              XOR는 다층으로 (NAND+OR→AND)
```

| 개념 | 한 줄 정의 |
|------|------|
| **One-Hot 인코딩** | 해당 자리만 1로 표현 |
| **퍼셉트론** | 가중합을 활성함수로 분류하는 뉴런 |
| **XOR 문제** | 단층 불가 → 다층으로 해결 |

퍼셉트론은 딥러닝의 뿌리다. 특히 **XOR 문제를 다층으로 해결**하는 아이디어가 오늘날 심층 신경망으로 이어졌다는 점이, 이산수학이 AI와 맞닿는 흥미로운 지점이다.
