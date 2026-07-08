---
title: "[시각화] 얼굴 인식, 닮은꼴 프로그램"
date: 2023-12-13 00:00:00 +0900
categories: [Bigdata-AI, Visualization]
tags: [visualization]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **네이버 Clova Face Recognition API**로 얼굴 인식 프로그램을 만든다. 사진 속 얼굴을 감지해 성별·나이·감정을 표시하고, 나아가 **유명인 닮은꼴**까지 찾아본다.

> **준비물** — 네이버 Developers에서 애플리케이션 등록 후 발급받은 **Client ID/Secret**, 그리고 `matplotlib`·`requests`·`json`. API URL은 얼굴 감지(`/face`)와 유명인 인식(`/celebrity`) 두 가지다.

---

## 1. 얼굴 감지 API 호출

```python
import matplotlib.pyplot as plt
import matplotlib.image as mpgimg
import json, requests

img = mpgimg.imread('familyPhoto.jpg')
plt.imshow(img); plt.show()

url = "https://openapi.naver.com/v1/vision/face"   # 얼굴 감지
files = {'image': open('familyPhoto.jpg', 'rb')}
headers = {'X-Naver-Client-Id': client_id, 'X-Naver-Client-Secret': client_secret}
response = requests.post(url, files=files, headers=headers)
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/2.png)

> ⚠️ **Client ID/Secret은 코드에 하드코딩하지 말자.** 환경 변수나 별도 설정 파일로 분리하고, Git에 올리지 않도록 `.gitignore`에 추가한다. 공개 저장소에 키가 노출되면 도용될 수 있다.

---

## 2. JSON 응답 파싱

```python
parsed = json.loads(response.text)   # 문자열 → 딕셔너리
print(parsed)
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/4.png)

> 💡 `response.text`는 읽기 힘든 긴 문자열이다. **`json.loads()`로 딕셔너리로 변환**하면 `parsed['faces']`처럼 키로 원하는 값을 뽑아낼 수 있다.

---

## 3. 얼굴에 정보 표시

감지된 각 얼굴에 **사각형 테두리 + 성별·나이·감정**을 그린다.

```python
import matplotlib.patches as patches

fig, ax = plt.subplots(figsize=(10,10))
ax.imshow(img)

for each in parsed['faces']:
    x, y, w, h = each['roi'].values()
    gender, gen_conf = each['gender'].values()
    age, _ = each['age'].values()
    emotion, _ = each['emotion'].values()

    rect = patches.Rectangle((x,y), w, h, linewidth=3, edgecolor='r', facecolor='none')
    ax.add_patch(rect)
    label = f"{gender} {gen_conf*100:.1f}%,\n{emotion},\n{age}"
    plt.text(x, y+h+50, label, size=8, color='blue')
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/6.png)

| 요소 | 역할 |
|------|------|
| `roi` | 얼굴 위치·크기(x,y,w,h) |
| `patches.Rectangle` | 인식 범위 사각형 |
| `plt.text` | 성별·나이·감정 라벨 |

작동은 잘 되지만, 오른쪽 남성이 99% 신뢰도로 여성으로 나오기도 했다("0.94%의 사나이"). API의 한계도 관찰된다.

---

## 4. 유명인 닮은꼴

URL만 `/celebrity`로 바꾸면 된다. 얼굴이 하나면 for문 없이 인덱스로 접근한다.

```python
url = "https://openapi.naver.com/v1/vision/celebrity"   # 유명인 인식
...
cel = parsed['faces'][0]['celebrity']['value']
cel_conf = parsed['faces'][0]['celebrity']['confidence']
```

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/7.png)

> 💡 결과가 **한글이면 matplotlib에서 깨질 수 있다.** 한글 폰트를 설정하거나, 간단히 `print()`로 콘솔에 출력해 확인한다(결과는 '이성경'!).

---

## 📝 정리

```
얼굴 인식 프로그램
├─ API     Clova Face Recognition(/face, /celebrity)
├─ 호출     requests.post(url, files, headers)
├─ 파싱     json.loads → parsed['faces']
└─ 표시     patches.Rectangle + plt.text
```

| 개념 | 한 줄 정의 |
|------|------|
| **Clova Face** | 네이버 얼굴 인식 API |
| **roi** | 얼굴 위치 정보 |
| **json.loads** | 응답 → 딕셔너리 |

핵심은 **API로 얼굴을 분석하고, 응답 JSON을 파싱해 이미지 위에 시각화**하는 것이다. URL만 바꿔 얼굴 감지·유명인 인식을 모두 처리할 수 있으며, API 키는 반드시 코드 밖으로 분리하자.
