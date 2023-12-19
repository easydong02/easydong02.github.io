---
title: "[시각화] 얼굴 인식, 닮은꼴 프로그램"
date: 2023-12-13 00:00:00 +0900
categories: [Bigdata-AI, Visualization]
tags: [visualization]
render_with_liquid: false
future: true
---

안녕하세요! 주말이 끝나고 다시 월요일이네요~ 힘내서 열심히 해봅시다!

이번에는 네이버 디벨로퍼에서 제공하는 얼굴인식 오픈API를 이용해서 한번 파이썬으로 그 프로그램을 만들어보겠습니다!

먼저 네이버 디벨로퍼에 들어가서 로그인을 하고 어플리케이션 등록을 하면 클라이언트 id와 비밀번호를 부여합니다. 그것을 가지고 오른쪽 상단에 serch here에서 face라는 키워드로 검색하면 저렇게 Clova Face Recognition이라는 api가 나오고 들어가서 구현예제를 누르면?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/1.png)

이런식으로 각 프로그래밍 언어에 대한 코드가 나옵니다. url은 2가지가 있는데 얼굴감지와 유명인 얼굴인식 2개가 있습니다 2개 해볼건데 먼저 그냥 얼굴감지인 ~/face 로 된 url을 쓰겠습니다!

```
import matplotlib.pyplot as plt
import matplotlib.image as mpgimg
import json #json 형식 사용 key : value
import requests # 인터넷 연결

img = mpgimg.imread('familyPhoto.jpg')   #사진 이름 설정
plt.figure(figsize=(10,8))          #액자설정
plt.imshow(img)                     #사진읽기
plt.show()                          #사진 쇼



client_id = 'YJ86dBbvZepeV6kbISPC'
client_secret = 'KgSdEC3k6r'

url = "https://openapi.naver.com/v1/vision/face" # 얼굴감지
files = {'image': open('familyPhoto.jpg','rb')}
headers = {'X-Naver-Client-Id': client_id, 'X-Naver-Client-Secret': client_secret }
response = requests.post(url,  files=files, headers=headers)
```

먼저 matplotlib에서 pyplot이랑 image를 임포트했습니다. 각각 그림과 사진을 담당합니다. 그리고 img라는 객체를 만들어서 미리 다운 받고 같은 경로에 넣어주었습니다. 그래서 matplotlib.image에 있는 imread메서드의 인자로 사진파일이름을 확장자까지 넣어주었습니다. pyplot에 있는 figure(),imshow(),show()로 액자설정과 사진을 읽고 출력합니다. 그리고 변수 2개를 만들어서 id 와 비밀번호를 넣었습니다~나머진 구현예제를 response 부분까지 그대로 긁어왔어요~ 실행해볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/2.png)

잘 나오네요! 사진은 구글에서 가족사진이라는 키워드로 검색해서 나왔습니다~

```
parsed =json.loads(response.text) # 딕셔너리로 전환
print(parsed)
```

위에서 사진을 분석한 정보가 response로 들어왔는데요 여기서 response.text로 객체정보를 볼 수 있습니다.. 그런데?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/3.png)

너무 가독성이 좋지 않군요! 그래서 미리 임포트한 json으로 parsed 라는 객체를 만들어서 json.loads(response.text)라는 문장으로 하나의 딕셔너리의 형태로 전환합니다. 그리고 parsed를 출력하면?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/4.png)

여전히 길지만 이제 여기서 키값을 가져와서 원하는 밸류를 뽑아내겠습니다~

```
import matplotlib.patches as patches

img = mpgimg.imread('familyPhoto.jpg')

fig, ax = plt.subplots(figsize =(10,10))
ax.imshow(img)


for each in parsed['faces']:
    x,y,w,h=each['roi'].values()
    gender,gen_conf = each['gender'].values()
    age,age_conf = each['age'].values()
    emotion,emo_conf = each['emotion'].values()

    rect_face =patches.Rectangle((x,y),w,h,linewidth=3,edgecolor='r',facecolor='none')
    ax.add_patch(rect_face)
    letter = gender +str(gen_conf*100)[0:5]+'%, \n'+emotion +', \n'+ age
    plt.text(x,y+h+50,letter,size=8,color='blue')
    print(gender+age)
```

자 이제 분석한 사진을 토대로 원하는 정보를 가져와 사진에 표시해볼게요! 먼저 matplotlib.patches를 임포트에서 사진에 우리가 원하는 정보를 붙일게요~ fig와 ax라는 변수를 만들어 subplots()에 사이즈로 (10,10)을 넣었습니다 이게 뭘까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/5.png)

리턴에 Figure랑 ax(축)을 반환하군요 이것으로 어떤 레이아웃을 더 편하게 만들 수 있겠어요!(사실 잘 모르지만..)

그 다음 for문으로 parsed에 있는 faces키값을 받아서 각 얼굴별 정보를 가져왔습니다. 이중포문이라 faces 안에도 또 여러키가 있었습니다! 또 거기서 나이라던지 성별이라던지 감정 등등.. 가져올 수 있겠죠! 그리고 각 얼굴을 어떻게 인식했는지 확인하기 위해서 rect\_face라는 변수를 만들어서 빨간색의 사각형 테두리를 인식범위에 설정하였습니다. 그리고 이것을 ax.add\_patch()의 인수로 넣어 붙였습니다. 그리고 글자도 사진에 붙일 건데요 각 얼굴의 주인마다 그 정보를 알기위해 성별과 그 신뢰도, 감정과 나이를 넣었습니다. 그리고 plt.text()의 인자로 넣고 위치도 조정을 했습니다! 실행해볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/6.png)

우와.. 정말 놀랍네요.. 물론 얼굴인식 프로그램은 제가 만들진 않았지만 대단하네요! 작동은 잘하는데 ..ㅋㅋ 오른쪽에 있는 남자분 여자로 나왔네요.. 심지어 99.06%의 신뢰도라.. 0.94%의 사나이입니다! 아무튼 재밌네요! 이번엔 유명인 닮은꼴 프로그램도 해보겠습니다! 다른점은 url 주소변경과 키값도 다를테니 그점만 유의하면 가능할것 같습니다!

```
import matplotlib.pyplot as plt
import matplotlib.image as mpgimg
import json #json 형식 사용 key : value
import requests # 인터넷 연결

img = mpgimg.imread('person.jpg')   #사진 이름 설정
plt.figure(figsize=(10,8))          #액자설정
plt.imshow(img)                     #사진읽기
plt.show()                          #사진 쇼



client_id = 'YJ86dBbvZepeV6kbISPC'
client_secret = 'KgSdEC3k6r'

url = "https://openapi.naver.com/v1/vision/celebrity" # 얼굴감지
files = {'image': open('person.jpg','rb')}
headers = {'X-Naver-Client-Id': client_id, 'X-Naver-Client-Secret': client_secret }
response = requests.post(url,  files=files, headers=headers)
```

일단 저는 한명의 닮은 꼴을 할거라 다른 사진을 구했습니다. 따라서 사진이름과 url만 바뀌고 나머진 똑같아요~

```
import matplotlib.patches as patches

img = mpgimg.imread('person.jpg')

fig, ax = plt.subplots(figsize =(10,10))
ax.imshow(img)

cel= parsed['faces'][0]['celebrity']['value']
cel_conf = parsed['faces'][0]['celebrity']['confidence']
# rect_face =patches.Rectangle(x,y,linewidth=3,edgecolor='r',facecolor='none')
# ax.add_patch(rect_face)
letter = cel +str(cel_conf*100)+'%, \n'

plt.text(550,250,letter,size=12,color='blue')
```

일단 읽어온 이미지 파일을 img변수에 넣고 여기서 인식된 얼굴은 하나라 for문을 쓰지 않고 그냥 인덱스번호로 연예인 결과과 그 신뢰도를 각각 변수에 넣어주었습니다. 그리고 나머진 같은 방식으로 하였습니다 결과를 볼까요?

![Desktop View](/assets/img/Bigdata-AI/Visualization/Face/7.png)

잘 나오네요! 그런데 한글이 깨집니다.. 그래서 제가 따로 print()를 이용해서 해봤습니다. 결과는 '이성경'님이시네요 ^~^

이번시간엔 여기까지 하겠습니다!