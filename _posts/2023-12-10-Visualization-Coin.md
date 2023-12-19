---
title: "[시각화] 파이썬으로 가상화폐 정보 조회하기"
date: 2023-12-10 00:00:00 +0900
categories: [Bigdata-AI, Visualization]
tags: [cryptocurrency, coin]
render_with_liquid: false
future: true
---

오늘은 재밌는걸 했습니다! 바로 파이썬으로 가상화폐 정보를 가져오는 것이죠! 여기에는 qt designer도 써봤습니다! 좀 어렵지만 이해한 만큼 분석해보겠습니다~

하기전에 먼저 코인거래소 (bithumb)정보를 가져와야 했습니다. 그래서 콘솔창(코딩창x)에 cip install pybithumb을 입력해서 거래소 정보를 연결했습니다!

qt1.py

```
import sys
from PyQt5.QtWidgets import *
from PyQt5 import uic
import pybithumb

form_class = uic.loadUiType("mywindow.ui")[0]

class MyWindow(QMainWindow, form_class):
    def __init__(self):
        super().__init__()
        self.setupUi(self)
        self.pushButton.clicked.connect(self.inquiry)  #이벤트 처리

    def inquiry(self):
        price = pybithumb.get_current_price("BTC")
        price2 = pybithumb.get_current_price("ETH")
        self.lineEdit.setText(str(price))
        self.lineEdit_2.setText(str(price2))


app = QApplication(sys.argv)
window = MyWindow()
window.show()
app.exec_()
```

먼저 uic를 임포트했습니다 uic로 ui파일을 불러올 수 있습니다. ui파일은 뭘까요? qt designer의 파일 형식입니다. 보통 우리는 gui를 만들 때 다 일일이 코드로 짰는데.. 이걸 사용하면 편합니다. 저는 파이썬을 아나콘다로 깔았는데 설치경로에 이게 있습니다~

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/1.png)

이렇게 코드가 아닌 샘플 형식으로 여기서 갖고 놀 수 있습니다.. 정말 편하죠?이것을 연결만 하면 됩니다!

마저 하자면, MyWindow 클래스를 만들고 여기에 QmainWindow(메인프레임), uic를 연결할 파일을 설정한 정보가 있는 변수 form_class를 상속받았습니다! 그리고 생성자에 setupUi()메서드와 이벤트 연결(자바의 액션 리스너와 같은?)해서 클릭할 때 이 함수의 inquiry()메서드를 실행하도록 했습니다. 그리고 그 메서드엔

```
price = pybithumb.get_current_price("BTC")
```

라는 문장이 있는데 어떤 뜻일까요? 빗썸에서 가져온 자료 중 현재의 가격을 가져오는 get_current_price()함수의 매개변수에 "BTC"를 넣었네요. 이것은 .. 맞습니다 그 비트코인입니다. 마찬가지로 ETH, 이더리움도 가져와서 price2에 넣었습니다. 그리고 lineEdit에 setText()로 그 값을 담았는데요 lineEdit은 무엇일까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/2.png)

이런 식으로 qt designer에는 어떤 컴포넌트를 누르면 그것에 대한 객체정보가 나옵니다. 그 이름중에 텍스트라인입니다!

```
app = QApplication(sys.argv)
window = MyWindow()
window.show()
app.exec_()
```

마지막으로 만들어진 MyWindow()클래스를 호출하는 메서드 show()와 프로그램 실행 코드를 작성하였습니다. 실행해볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/3.png)

우와.. 가상화폐의 현재가격이 출력되었습니다! 재밌네용 ㅎㅎㅎㅎㅎ

그런데 뭔가 부족합니다. 보통 현재가는 실시간으로 알려줘야하는데 저 조회버튼을 계속 누르고 있을 순 없습니다! 그래서 계속 새로고침을 하게 만들어보겠습니다.

```
import sys
from PyQt5.QtCore import QTime, QTimer
from PyQt5.QtWidgets import *
from PyQt5 import uic
import pybithumb

form_class = uic.loadUiType("mywindow2.ui")[0]

class MyWindow(QMainWindow, form_class):
    def __init__(self):
        super().__init__()
        self.setupUi(self)

        self.timer = QTimer(self)
        self.timer.start(1000)
        self.timer.timeout.connect(self.inquiry)

    def inquiry(self):
        cur_time = QTime.currentTime()
        str_time = cur_time.toString("hh:mm:ss")
        self.statusBar().showMessage(str_time)

        price = pybithumb.get_current_price("BTC")
        price2 = pybithumb.get_current_price("ETH")
        self.lineEdit.setText(str(price))
        self.lineEdit_2.setText(str(price2))


app = QApplication(sys.argv)
window = MyWindow()
window.show()
app.exec_()
```

다시 만들어봤습니다 처음 작성한거랑 어떤 부분이 같고 다른지 알아볼게요!

```
        self.timer = QTimer(self)
        self.timer.start(1000)
        self.timer.timeout.connect(self.inquiry)
```

타이머라는 메서드를 썼군요. QTimer클래스의 객체를 만들었습니다. start()는 인수로 지정한 밀리초 단위만큼 반복할 겁니다. 그리고 그 반복할때 바로 inquiry()를 호출합니다! 전에는 버튼을 눌렀을 때였는데 이번엔 1초마다 실행이 되겠군요 ㅎㅎ

```
        cur_time = QTime.currentTime()
        str_time = cur_time.toString("hh:mm:ss")
        self.statusBar().showMessage(str_time)
```

inquiry()메서드안에 있는 코드입니다. 현재 시간을 cur_time에 넣고 그것을 문자열로 바꾼 뒤 str_time에 넣고 그것을 statusBar, 상태바에 나타내도록 하였습니다! 실행해볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/4.png)

조회 버튼을 없애고 자동 실행에 좌측 하단에 현재시간이 나옵니다~ 이제 계속 가상화폐의 현재가를 가져오겠죠?

이제는 현재가만 아니라 여러가지를 뽑아볼게요

```
from PyQt5.QtCore import QDateTime
from PyQt5.QtWidgets import QDateTimeEdit
import pybithumb
import time
import datetime



orderbook = pybithumb.get_orderbook("BTC")
print(orderbook)
```

orderbook이라는 변수에 빗썸에 있는 BTC의 orderbook을 가져왔습니다. 뭐가 출력될까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/5.png)

엄청많네요... 일단 우리가 알 수 있는건 이것이 키값과 밸류로 이루어진 딕셔너리라는 것이죠! 그중에 현재 주문에 대한 정보를 다 담고있습니다!

```
orderbook = pybithumb.get_orderbook("BTC")

orderbook['bids']

for bid in bids:
    print(bid)
```

그중 bids키를 넣어서 밸류값을 출력하겠습니다 bids는 매수입니다. 그리고 for문으로 그 안의 원소들을 출력해볼게요!

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/6.png)

현재 가격과 거래량이 주루룩 나옵니다~

```
from pandas.core.window.rolling import Window
import pybithumb

df = pybithumb.get_ohlcv("BTC")

ma5=df["close"].rolling(window=5).mean()
last_ma5=ma5[-1]

price = pybithumb.get_current_price("BTC")

if price> last_ma5:
    print("상승장")
else:
    print("하락장")
```

이 것은 무엇일까요? 일단 df라는 변수에 비트코인에 대한 get_ohlcv()메서드가 있군요! 이것은 시가,고가,저가,종가,거래량을 보여줍니다 ! 이것도 딕셔너리의 형식입니다 이중 c에 해당되는 close키값은 그 화폐의 모든 날의 종가를 보여줍니다. 그래서 df\['close'\]는 그것을 담고 있겠네요. 그런데 그 뒤에 rolling(window=5).mean()이 보이네요? 이것은 모든 날의 이전 5일간의 평균입니다. 따라서 종합하면 ma5에는 비트코인의 모든날의 5일간의 종가평균을 보여주겠네요! 그리고 last_ma5=ma5\[-1\]는 그 모든날의 마지막 리스트 원소니까 오늘이겠죠? 그리고 price에는 비트코인의 현재가를 담았습니다. 그래서 현재가가 지난 5일간의 평균보다 높으면 상승장, 아니면 하락장을 출력하도록했습니다! 실행해볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/7.png)

하락장이네요 ㅎㅎ;

이제는 하나의 종목 말고 전체 장을 볼까요? 한 종목당의 상승장, 하락장을 결정한 다음 상승장이 하락장보다 많으면 전체 상승장, 아니면 전체하락장으로 출력하려고합니다!

```
from pandas.core.window.rolling import Window
import pybithumb


def all(ticker):
    df = pybithumb.get_ohlcv(ticker)
    ma5=df["close"].rolling(window=5).mean()
    last_ma5=ma5[-2]

    price = pybithumb.get_current_price(ticker)

    if price> last_ma5:
        return True
    else:
        return False

tickers = pybithumb.get_tickers()
tup =0
tdown = 0
for n in tickers:
    if all(n):
        tup+=1
    else:
        tdown +=1

if tup>tdown:
    print("전체상승장")
else:
    print("전체하락장")
```

함수를 하나 만들었습니다. all이란 이름입니다. 여기선 ticker가 한 종목입니다. 따라서 get_tickers()는 전체 종목 이름을 갖고있겠죠?

아까 만든 로직과 비슷합니다. 하지만 이 함수는 그 결과를 상승장이면 True, 하락장이면 False를 반환합니다. 따라서 이 함수의 매개변수에 코인 이름을 넣으면 그 값이 반환되겠죠?

그 다음엔 모든 종목을 tickers변수에 담았습니다. 그래서 for문으로 tickers의 원소 하나하나를 all()함수에 넣어 상승장이면 tup을 1증가 시키고 하락장이면 tdown을 1증가시킵니다. 그리고 마지막에 그 수를 비교해서 전체상승인지 하락인지 판단해보겠습니다!

![Desktop View](/assets/img/Bigdata-AI/Visualization/Coin/8.png)

시간이 좀걸렸네요 모든 코인을 조사해야하니까 ㅎㅎ 결국 전체하락장으로 판명났습니다 ㅎㅎ 돔황챠!!

이번엔 여기까지 하겠습니다!!
