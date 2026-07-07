---
title: "[Python] 크롤링(Crawling)"
date: 2023-11-17 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **크롤링(Crawling)**으로 웹 데이터를 수집한다. 다음 뉴스 헤드라인과 로또 번호를 가져와 데이터프레임으로 만들고 시각화까지 해본다.

> **크롤러(crawler)**: 자동화된 방법으로 웹을 탐색하는 프로그램. 웹을 훑는 것이 **웹 크롤링**, 그 데이터를 수집하는 것이 **데이터 크롤링**이다.

**크롤링의 기본 흐름:**

```
① URL 정의 → ② requests.get()으로 요청 → ③ BeautifulSoup으로 HTML 변환 → ④ select()로 원하는 태그 선별
```

---

## 1. 뉴스 헤드라인 크롤링

```python
# 다음 뉴스 페이지 크롤링
url = 'https://news.daum.net/'

resp = requests.get(url)                              # ② 요청
html = BeautifulSoup(resp.text, 'html.parser')        # ③ HTML 변환

for news in html.select('a.link_txt')[:-13]:          # ④ 태그 선별
    print(news.text.strip().replace('[', '').replace(']', ''))
```

![Desktop View](/assets/img/Programming-Language/Python/Crawling/1.png)

> 💡 개발자 모드로 원하는 요소의 태그(`a.link_txt`)를 확인해 `select()`에 넣는다. `.text.strip()`으로 텍스트만 추출하고 `.replace()`로 불필요한 문자를 정리한다.

---

## 2. 로또 번호 크롤링 (1~30회차)

```python
lotto_list = []
for i in range(1, 31):
    print(f'{i}회차 크롤링 중입니다.')

    # 차단 방지 (랜덤 딜레이)
    seed = np.random.randint(100)
    np.random.seed(seed)
    time.sleep(np.random.randint(5))

    url = f'https://search.daum.net/search?w=tot&DA=LOT&q={i}회차%로또'
    resp = requests.get(url)
    html = BeautifulSoup(resp.text, 'html.parser')

    for ball_nm in html.select('span.ball')[:6]:
        lotto_list.append(int(ball_nm.text))

print('크롤링 완료!!!')
```

> ⚠️ 반복 요청 시 **차단(block)을 피하기 위해** 랜덤 딜레이(`time.sleep`)를 준다. 모든 번호를 **하나의 리스트**에 담는 이유는, 이후 `reshape`으로 데이터프레임을 만들기 위함이다.

---

## 3. 데이터프레임 변환

```python
lotto = np.array(lotto_list).reshape(-1, 6)   # 6개씩 묶어 회차별 행으로
index_list = [f'{i}회차' for i in range(1, 31)]
df = pd.DataFrame(lotto, index=index_list)
df
```

![Desktop View](/assets/img/Programming-Language/Python/Crawling/2.png)

> 💡 `reshape(-1, 6)`은 "열은 6개로 고정, 행은 자동 계산"이라는 뜻. 리스트를 회차별 6개 번호의 표로 재구성한다.

---

## 4. 시각화

```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.figure(figsize=(12, 6))
sns.countplot(lotto_list)   # 번호별 출현 빈도
plt.show()
```

![Desktop View](/assets/img/Programming-Language/Python/Crawling/3.png)

`countplot`으로 각 번호의 빈출을 확인할 수 있다.

> 💡 번호별 편차가 큰 것은 표본이 30회차뿐이기 때문이다. 회차가 길어질수록 **통계적 확률(균등)**에 가까워질 것이다.

---

## 📝 정리

```
크롤링 흐름
├─ 요청       requests.get(url)
├─ 파싱       BeautifulSoup(resp.text, 'html.parser')
├─ 선별       html.select('태그')
├─ 차단 방지  time.sleep()으로 랜덤 딜레이
└─ 가공/시각화  reshape → DataFrame → countplot
```

| 도구 | 역할 |
|------|------|
| **requests** | URL에 데이터 요청 |
| **BeautifulSoup** | HTML 파싱 |
| **select()** | CSS 선택자로 태그 추출 |
| **reshape/DataFrame** | 수집 데이터를 표로 |

크롤링은 **"요청 → 파싱 → 선별"**의 단순한 3단계 흐름이다. 여기에 차단 방지(딜레이)와 데이터 가공(DataFrame)을 더하면 실용적인 데이터 수집 파이프라인이 완성된다.
