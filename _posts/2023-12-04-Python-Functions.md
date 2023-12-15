---
title: [Python] 여러가지 함수들
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이번엔 파이썬으로 배운 여러가지 함수를 리뷰해보겠습니다!!

\# 변수명 짓기

첫문자는 문자 또는 언더바 이어야 합니다. 자바랑 비슷하네용 \_\_init\_\_ \_\_name\_\_ \_\_main\_\_ 언더바 2개로 시작하는 이름 특별한 의미를 갖습니다. 아직 배우진 않았지만.. 그래서 언더가 2개로 시작하는 이름은 사용하면 안 됩니다~ 이미 사용되는 키워드는 피합시다 (if, else, for 등등..)

sep = 문자열 : 문자열 구분자를 기본 설정인 빈칸에서 다른 문자로 설정합니다.

sep = ''와 같이 설정하면 구분자가 없는 것으로 설정합니다~

end = 문자열 : 마지막 인수를 출력한 후 맨 마지막에 출력할 문자열을 설정합니다. 기본은 줄바꿈, 즉 개행문자입니다. 줄 넘김을 하고 싶지 않으면 end=''와 같이 빈문자를 입력하시면 되겠습니다.

```
side1 = float(input('한변의 길이를 입력하세요'))
side2 = float(input('다른 한 변의 길이를 입력하세요'))

hyp = ((side1 **2) + (side2 ** 2)) **0.5

print('빗변의 길이 : ',hyp ,end =' ')
print("입니다.")
```

피타고라스 정의를 이용해서 삼각형의 빗변의 길이를 구하는 코드입니다~

```
def fibo(n):
    a,b =1,0
    while a<=n:
        print(a,end=" ")
        a,b = a+b,a


fibo(5)
```

이것은 피보나치 수열을 표현한 함수입니다. 0,1,1,2,3,5,8 ... 이런식으로 계속 증가하는 수열이고 a가 n보다 커질때까지 반복합니다~ 이문제는 코드는 간결하면서 생각하는 연습을 하게 해준 문제였습니다 ㅎㅎ

```
from random import randint

n = randint(1,500)

while True:
    ans = int(input("어떤 숫자일까요?"))
    if ans >n :
        print("Down")
    elif ans<n:
        print("Up")
    else:
        print("정답!")
        break
```

난수입니다. random 모듈에 있는 randint()라는 함수는 인수의 범위 사이에 무작위로 한 값을 선택합니다. 사용자가 입력한 숫자를 토대로 선택된 난수보다 크고작음을 이용해서 up down 게임을 만들었습니다. ㅋㅋ 재밌네요

**문자열**

```
mystr = "문자열은 불변"
print(id(mystr))
mystr = "숫자열은 불변"
print(id(mystr))
```

같은 객체지만 문자열은 수정이 안됩니다. 따라서 바뀌면 새로운 주소값을 부여하고 그 주소값을 객체에 넣죠. 따라서 위와같이 내용이 변경되었을 경우 id()함수로 주소값을 출력하면 서로 다른 주소값을 나타냅니다~

```
a = "RoboCop"
b= a[1::2]
print(b)
```

처음보는 슬라이싱이네요. 1::2는 중간에 끝을 나타내는 인덱스 인자가 없다는 것입니다. 따라서 끝까지 출력하겠죠. 그런데?! 마지막2는 step입니다. 따라서 한칸을 건너뛰고 하나씩 출력하죠. 그래서 b를 출력하면 ooo가 나옵니다. ㅎㅎ

```
def reverse_str(s):
    return s[::-1] # start stop step

reverse_str("12345")
```

문자열을 거꾸로 배치하는 함수를 만들었습니다. step 칸에 -1을 하면 뒤에서부터 하나씩 시작하게됩니다.

문자열은 불변인데.. 그렇다면 문자열을 갖고 놀 수는 없을까요?

```
a = "Big"
a = a+"Bad"
a = a+ "John"
```

저는 Big Bad John이라는 문자열을 만들고 싶어서 하나씩 저렇게 추가를 했습니다. 과연이게 수정일까요? 아닙니다. 하나씩 추가될 때마다 a에는 새로운 주소값이 배정되고 있죠.. 그런데 리스트와 join()함수를 이용하면 가능합니다!

```
n = ord('A')
s = []
for i in range(n,n+26):
    s.append(chr(i))

print(s)

l = ''.join(s)
print(l)
```

A부터 Z까지 출력하는 반복문을 만들었습니다. 여기서 s는 리스트로 append()함수를 통해서 하나씩 값을 집어 넣을겁니다. 리스트는 변경이 가능하기 때문! 그런데 그냥 넣기만 하고 출력하면 \['A','B', ...'Z'\]이런식으로 하나씩 들어갑니다. 그러기 위해선 join()으로 리스트들의 원소들을 간격없이 쭈욱 재배치하도록 했습니다.

**n진수 표현법.**

```
n = int('10001' ,2) #10001을 2진수로 인식후 10진수로 변환
print(n)
```

이것은 10001이라는 것을 그냥 출력하면 10진수로 나오겠죠? 그런데 저렇게 ,2로하면 이것을 이진수로 인식합니다. 따라서 저 n을 출력하면 17이 나옵니다.

**is함수들**

```
str= '$$123&gf'
str.isalnum()
```

is함수들은 전부 논리값 결과를 반환합니다. 그중에서 isalnum은 안에 문자열이 글자와 숫자로만 이루어져 있는지 검사합니다.

```
str = 'awdawd'
str.isalpha()
```

isalpha()함수는 문자열이 알파벳으로만 이루어져 있을 때 True를 반환합니다.

```
str = '6548932'
str.isdecimal()
```

isdecimal()은 10진수 숫자로만 이루어져 있는지 검사합니다~

```
str = 'asefsf'
str.islower()
```

islower()는 문자열이 전부 소문자인지 판단합니다. 반대로 isupper()는 대문자겠죠?

```
str = ' '
str.isspace()
```

isspace()는 문자열값이 전부 공백인가를 판단합니다.

```
str = 'Smith'
str.istitle()
```

이것은 첫글자만 대문자인지를 판단합니닷!

```
str = 'Sir. John Bennett. PhD'
is_doc = str.endswith('PhD')
print(is_doc)
is_Sir = str.startswith('Sir')
print(is_Sir)
```

이것은 endswith()와 startswith()이라는 함수인데요. 매개변수 안에 있는 문자열로 시작하거나 끝나면 True를 반환합니다~

```
str_list = 'Moe Larry   Curlyu  Shemp'.split()
str_list
```

split()은 저렇게 되어있는 문자열을 공백은 무시하고(매개변수에 아무것도 안 넣었으면) 각각의 기준점들을 리스트의 요소로 담습니다!

![Desktop View](/assets/img/Programming-Language/Python/Functions/1.png)

이렇게 나옵니다~

```
str_list = 'Moe Larry     Curlyu    Shemp'.split(' ') 
str_list
```

매개변수에 공백을 넣어주면 하나의 공백만 문자로 인정합니다. 따라서 리스트에 ' '값이 있겠네요~

```
#strip  lstrip  rstrip 여백을 제거
name ='      Will Shakes             '
new_name= name.strip()
new_name
```

strip()은 문자열(공백제외) 시작과 끝 기준으로 그 바깥에 있는 공백을 전부 지워줍니다.

비슷하게 rstrip()과 lstrip()이 있는데 오른쪽과 왼쪽 공백을 없앱니다\`

이번시간에는 여기까지 하도록 하겠습니당~