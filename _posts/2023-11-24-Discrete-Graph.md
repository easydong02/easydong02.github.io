---
title: [이산수학] 그래프
date: 2023-11-24 00:00:00 +0900
categories: [Math, 이산수학]
tags: [math, discretemath, graph]
render_with_liquid: false
future: true
---

오늘은 그래프와 순전파, 역전파를 알아보겠습니다!

### **그래프**

정점 vertex의 집합 V와 서로 다른 정점쌍 (vi,vj)를 연결하는 모서리 edge의 집합 E로 구성된 구조입니다.

![Desktop View](/assets/img/Math/Discrete-Math/Graph/1.png)

#### **그래프 용어**

**인접adjacent**

정점 v 와 다른 정점 v2를 연결하는 모서리가 존재할 때 v와 v2는 인접합니다.

**근점incident**

모서리 e는 정점 v와 v2를 연결하면 모서리 e는 점정 v와 v2에 근접합니다.

 **N(v)**

정점 v의 이웃 정점 집합

**차수degree**

d(v): 정점 v에 근접하는 모서리의 개수

루프는 두번으로 계산( 한쪽으로 나가서 다른 한쪽으로 들어오기 때문)

#### **그래프의 형태**

**단순그래프**

두 정점 사이에 오직 하나의 모서리가 있는 그래프

**다중그래프**

두정점 사이에 둘 이상의 모서리가 있는 그래프

**의사그래프**

루프를 허용하는 그래프

**가중치그래프**

모서리에 가중치가 부여된 그래프, 앞으로 인공지능에서 자주 보게 될 그래프입니다.

![Desktop View](/assets/img/Math/Discrete-Math/Graph/2.png)

#### **그래프의 표현**

그래프의 표현에는 대표적으로 각점을 행,렬의 인덱스로 구성하고 이어진 점을 인덱스로 하는 원소에 1을, 그렇지 않은 경우 0을 삽입하여 행렬로써 표현합니다.

![Desktop View](/assets/img/Math/Discrete-Math/Graph/3.png)

**뉴럴넷과 그래프**

![Desktop View](/assets/img/Math/Discrete-Math/Graph/4.png)

**계산 그래프**

![Desktop View](/assets/img/Math/Discrete-Math/Graph/5.png)

이번 시간에는 여기까지 하겠습니다. 다음엔 순전파와 역전파에 대해서 적어볼 예정입니다!