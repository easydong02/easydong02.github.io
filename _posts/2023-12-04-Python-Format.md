---
title: [Python] 패킹 포맷
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

여전히 돌아온 파이썬 시간! 오늘 배운거 적을게요! 아 그리고 원래하던 문제들도 있는데 안 올렸는데 오늘부터는 올리겠습니다!

```
n =20
greeting = '안녕하세요'
place ='뉴월드'
welcome ='환영합니다.'

print('{}번 손님 {}, {}에 오신것을 {}!'.format(n,greeting,place,welcome))
```

포맷입니다. 마치 C언어에서 %d를 처리하는 로직과 비슷하죠? 파이썬에서는 중괄호({})로 처리하고 그 문자열의 .format으로 각 중괄호에 원하는 값을 넣을 수 있습니다!

```
list2=[1,2,3]

list3 = list2+[16]
print(list3)
```

리스트에 원하는 값을 추가했습니다. 리스트의 형식을 지킨 원소를 +로 해서 기존 값에 추가를 했어요!

```
del list3[-1] # 맨마지막 인덱스
print(list3)
```

del 명령어입니다. 원하는 원소를 삭제할 수 있습니다. -1은 마지막 원소겠죠?

```
bts= ['RM','진','슈가','제이홉','지민','뷔','정국']
for i in enumerate(bts):
    print('{}'.format(i))
```

bts 멤버들을 문자열로 리스트에 넣었습니다! for문으로 모든 멤버들의 이름을 출력하려고 합니다. 그런데 enumerate()함수는 뭘까요? 바로 리스트들에 번호를 붙여줍니다! 따라서 그냥 i만 출력해도

![Desktop View](/assets/img/Programming-Language/Python/Format/1.png)

이런식으로 출력이 되네요! 재밌습니다 ㅎㅎ

```
import urllib.request

def getweb(url):
  response = urllib.request.urlopen(url)
  data = response.read()
  decoded = data.decode('utf-8')
  return decoded

url = input('웹페이지 주소를 입력하셔요')
print(getweb(url))
```

이것은 웹주소를 입력하면 그 웹페이지를 HTML형식으로 볼 수 있습니다. import로 urlib.request를 가져오고 uropen()메서드를 이용해 url을 response에 넣어주고 그걸을 다시 read()메서드 로 data에 넣어준 뒤 utf-8로 디코딩했씁니다. 저는 http://www.naver.com을 입력했습니다

![Desktop View](/assets/img/Programming-Language/Python/Format/2.png)

네이버 메인페이지는 예상했던것보다 엄청났군요..

```
# 딕셔너리와 튜플

#딕셔너리 : 키값과 밸류 값으로 구성, 키로 밸류 값을 조회
#           인덱싱과 슬라이싱 불가능
wintable ={
    "가위":'보',
    '보':'바위',
    '바위':'가위'
}

def rsp(mine,yours):
    if mine==yours:
        return "비겼다"
    elif wintable[mine]==yours:
        return "내가 이겼다"

    elif mine ==wintable[yours]:
        return "내가 졌다"

print(rsp(input(),"바위"))
```

딕셔너리를 이용해서 키값을 밸류값을 이기는 가위바위보로 설정했씁니다. 그리고 rsp 함수로 내가낸 패에 따라 승부를 출력하는 함수를 만들었습니다!

```
dict = {'one': 1,'two':2,'three':3}
dict['two'] = 22
dict['four']=4

print(dict)

del(dict['one'])
print(dict)

dict.pop('three')
print(dict)
```

또다른 딕셔너리입니다. del과 pop()함수로 리스트와 같이 딕셔너리도 이런식으로 수정이 가능합니다~

```
names ={'Jane':32, 'Tod':35,'Paul':62}

for i,j in names.items():
    print('{}의 나이는 {}입니다.'.format(i,j))
```

딕셔너리는 키값과 밸류값으로 나눠져있기 때문에 items()로 두개를 쌍으로 가져올 수 있습니다. 따라서 i,j로 둘다 가져올 수 있습니다!

간단하게는

```
names ={'Jane':32, 'Tod':35,'Paul':62}

for i in names.items():
    print('{}의 나이는 {}입니다.'.format(*i))
```

이렇게도 할 수 있습니다! \*는 저번에 배웠던 것처럼 복수의 변수를 설정할 때 사용합니다!

```
t1 = (1,2,3,4,5)
print(type(t1))
l1 = list(t1)

print(l1)
print(type(l1))

l1[0] = 6

print(l1)

t1 = tuple(l1)
print(t1)
```

튜플은 수정할 수 없는데, 리스트로 변환해준뒤 수정하고 다시 튜플로 변환하면 가능은 합니다 ㅎㅎ

```
c=(3,4)

d,e =c #언패킹해서 d와 e에 넣는다.

print(d)
print(e)

f=d,e #패킹
print(f)
```

튜플 c를 d,e로 나눠서 패킹했습니다. 그리고 그것을 다시 f에 넣어주면 다시 튜플로 패킹이됩니다!! 신기하네요.. 뭔가 파이썬은 우리가 생각하는 것을 다 문법으로 구현해놓은 느낌?..

```
list=[1,2,3,4,5]

for a in enumerate(list):
    print('{}번 값: {}'.format(*a))
```

이것은 무엇일까요? enumerate()함수를 이용해서 리스트를 인덱스번호와 값으로 \*기호를 이용해 출력했습니다!

![Desktop View](/assets/img/Programming-Language/Python/Format/3.png)

잘 나오네요 ^^

이번시간엔 여기까지 하겠습니다!!