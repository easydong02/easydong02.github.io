---
title: "[시각화] 파이썬으로 가상화폐 정보 조회하기"
date: 2023-12-10 00:00:00 +0900
categories: [Bigdata-AI, Visualization]
tags: [cryptocurrency, coin]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 파이썬으로 **가상화폐 거래소(빗썸) 정보를 조회**한다. Qt Designer로 만든 GUI에 현재가를 표시하고, `QTimer`로 실시간 갱신하며, 이동평균선으로 **상승장/하락장을 판별**하는 것까지 만들어본다.

> **사용 도구** — `pybithumb`(빗썸 API), `PyQt5`(GUI), Qt Designer(`.ui` 파일). 설치는 `pip install pybithumb`.

---

## 1. Qt Designer + 현재가 조회

`.ui` 파일(Qt Designer로 만든 화면)을 `uic`로 불러와 버튼 클릭 시 현재가를 표시한다.

```python
from PyQt5.QtWidgets import *
from PyQt5 import uic
import pybithumb

form_class = uic.loadUiType("mywindow.ui")[0]

class MyWindow(QMainWindow, form_class):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.pushButton.clicked.connect(self.inquiry)   # 버튼 이벤트 연결

    def inquiry(self):
        price = pybithumb.get_current_price("BTC")
        self.lineEdit.setText(str(price))
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/1.png)

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/3.png)

> 💡 **`.ui` 파일**은 Qt Designer로 GUI를 드래그&드롭으로 만든 결과물이다. 코드로 일일이 위젯을 배치하는 대신, `uic.loadUiType()`로 불러와 연결만 하면 되어 훨씬 편하다. `clicked.connect()`는 자바의 액션 리스너와 같다.

---

## 2. QTimer로 실시간 갱신

버튼을 계속 누를 순 없으니, **1초마다 자동 조회**하게 한다.

```python
from PyQt5.QtCore import QTime, QTimer

def __init__(self):
    ...
    self.timer = QTimer(self)
    self.timer.start(1000)                    # 1000ms마다
    self.timer.timeout.connect(self.inquiry)  # inquiry 반복 호출

def inquiry(self):
    str_time = QTime.currentTime().toString("hh:mm:ss")
    self.statusBar().showMessage(str_time)    # 상태바에 시간 표시
    price = pybithumb.get_current_price("BTC")
    self.lineEdit.setText(str(price))
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/4.png)

> 💡 **`QTimer`**는 지정한 밀리초마다 `timeout` 신호를 보낸다. 이를 `inquiry`에 연결하면 버튼 없이 1초마다 자동으로 현재가가 갱신된다.

---

## 3. 호가창(orderbook) 조회

```python
orderbook = pybithumb.get_orderbook("BTC")   # 딕셔너리
for bid in orderbook['bids']:                # bids=매수 호가
    print(bid)
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/6.png)

`get_orderbook`은 현재 주문 정보를 딕셔너리로 반환하며, `bids`(매수)·`asks`(매도)에 가격·수량이 담긴다.

---

## 4. 이동평균선으로 상승장/하락장 판별

```python
df = pybithumb.get_ohlcv("BTC")               # 시/고/저/종/거래량
ma5 = df["close"].rolling(window=5).mean()    # 5일 종가 이동평균
last_ma5 = ma5[-1]
price = pybithumb.get_current_price("BTC")

print("상승장" if price > last_ma5 else "하락장")
```

> 💡 **`rolling(window=5).mean()`은 5일 이동평균선(MA5)**이다. 현재가가 최근 5일 평균보다 높으면 상승 추세로 본다. 주식·코인 차트 분석의 기본 지표다.

**전체 장 판별** — 모든 종목을 순회해 상승/하락을 집계한다.

```python
def is_up(ticker):
    df = pybithumb.get_ohlcv(ticker)
    last_ma5 = df["close"].rolling(window=5).mean()[-2]
    return pybithumb.get_current_price(ticker) > last_ma5

tickers = pybithumb.get_tickers()
tup = sum(1 for t in tickers if is_up(t))
print("전체상승장" if tup > len(tickers) - tup else "전체하락장")
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/8.png)

> ⚠️ 모든 종목을 순회하며 API를 호출하므로 **시간이 오래 걸린다.** 종목 수만큼 네트워크 요청이 나가니, 실제 서비스라면 캐싱이나 배치 조회를 고려해야 한다.

---

## 📝 정리

```
가상화폐 조회
├─ GUI     Qt Designer(.ui) + uic로 로드
├─ 실시간   QTimer(1초마다 갱신)
├─ 데이터   get_current_price / get_orderbook / get_ohlcv
└─ 분석     MA5(5일 이동평균)로 상승/하락 판별
```

| 개념 | 한 줄 정의 |
|------|------|
| **pybithumb** | 빗썸 시세 API |
| **QTimer** | 주기적 자동 실행 |
| **MA5** | 5일 이동평균선 |

핵심은 **API로 시세를 받아 GUI에 실시간 표시**하고, **이동평균선으로 추세를 판별**하는 것이다. Qt Designer 덕분에 GUI를 쉽게 만들고, QTimer로 실시간성을 더해 간단한 시세 조회 프로그램을 완성했다.
