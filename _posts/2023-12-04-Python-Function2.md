---
title: "[Python] 함수 사용자입력 반복문 클래스"
date: 2023-12-04 00:00:00 +0900
categories: [Programming-Language, Python]
tags: [python]
render_with_liquid: false
future: true
---

이번에는 함수랑 클래스부분 맛만 보겠습니다.

```
# 커피 자판기 
# 남은 커피가 있으면 판매 -> 판매 -> 없으면 안 판매

coffee =10
while True:
  money =int(input("돈을 넣어 주세요 : "))
  if money == 300:
    print("커피를 드립니다.")
    coffee -=1
  elif money >300:
    print("거스름 돈은 %d원입니다." %(money-300))
    coffee -=1
  else:
    print("금액이 부족합니다.")
  
  if coffee ==0:
    print("매진되었습니다")
    break
```

반복문과 조건문으로 만들었습니다. 이렇게하면 입력한 값이 300이 초과할 경우 커피를 하나씩 빼고 출력문을 실행한 뒤 다 떨어지면 break로 반복문을 빠져나오게 했습니다!

**for문**

파이썬의 직관적인 특징을 잘 보여주는 것이 for문입니다. while문과 비슷하지만 for문은 구조를 살펴보기에 좋습니다!

```
list = [1,2,3,4,5]
for i in list:
  print(i)
```

자바에서는 int i 설정하고 범위와 증가형을 다 해주어야 했지만 파이썬에서는 저렇게 i in list를 쓰면 i는 list의 원소값을 처음부터 하나씩 끝까지 가져올 때까지 반복합니다! 정말 간단하죠?

```
a = [(1,2),(3,4),(5,6)]

for x in a:
  print(x)
```

이렇게 원소값을 하나의 튜플로서 해줘도 반복해줍니다. 따라서 x에는 (1,2) , (3,4), (5,6) 이 세트로 하나씩 대입이 됩니다!

```
a = [(1,2),(3,4),(5,6)]

for x,y in a:
  print(x+y)
```

이렇게 x,y로 a에서 저 튜플을 한개씩으로 보는게 아니라 각각의 값을 추출할 수 있습니다. 따라서 출력문은 3,7,11이 나옵니다!

```
score = [90,70,50,40,80]
i=0
for x in score:
  i +=1
  if x<60:continue

  print("%d번 합격" %i)
```

score의 원소를 하나씩 x에 담은 후 반복을 할때마다 i를 1씩 증가시킵니다. 그리고 x가 60점 미만이라면 continue로 그 다음 반복을 실행합니다!

따라서 60점 이상인 분들만 출력이 되겠죠?

```
add =0

for i in range(1,11,2):
  add+=i

print(add)

add=0
for i in range(11,1,-2):
  add+=i

print(add)
```

\# range함수와 for 문

\# range(start, stop, step)start를 생략하면 0부터, step을 생략하면 1씩 증가합니다. 이렇게 1부터 11미만까지(10까지) 2씩 증가하거나 -2씩 증가, 즉 2씩 감소하게해서 i에 넣었군요!

```
# for 문과 range를 이용해서 구구단을

for i in range(2,10):
  print(" ")
  for j in range(1,10):
    print(i*j,end="")
```

range문으로 구구단을 만들었습니다! 자바보다 훨씬 편하네요~

**List Comprehension**

for문을 사용하면 여러 줄을 작성할 것을 한번에 할 수 있습니다.

```
a = [1,2,3,4]
result = [n*3 for n in a if n%2 ==0]
print(result)
```

result 부분에 보시면 a라는 리스트에서 하나씩 n으로 집어넣습니다. 그리고 그 n에 3을 곱한것을 result에 넣어주죠. 단 조건문도 넣을 수 있어서 저는 짝수를 설정했습니다. 조건은 더 n이 최종적으로 연산이 되고 검사를 하는건가 봅니다!

```
i =0


while i<5:
  i +=1
  for i in range(1,i+1):
    print("*",end ="")
  print("")
```

항상 코딩문제 나오면 등장하는 기본문제.. \*출력입니다. 하나씩 증가하면서 \*을 출력합니다!

함수란?

과일 -> 믹서기 -> 과일주스 바나나 -> 믹서기 -> 바나나주스 사과 -> 믹서기 -> 사과주스

왜! 항상 반복되는 코드를 매번 작성하는 것보다 그것들을 묶어서 함수로 만들면 쉽고 편하게 재활용 할 수 있습니다!

문법은

기본형

def 함수명(변수): 수행 수행 수행 return 값

def add(d,b): return a+b

이 함수의 이름은 add이고 입력으로 2개의 값을 받아서 더해주는 함수입니다. 함수에는 매개변수가 있을 수도, 없을 수도있습니다. 리턴값도 마찬가지로요.. 그러면 함수의 종류에는 4개가 있겠네요!

```
#입력이 몇개가 될지 모를 때!!

def add_many(*args):
  result =0
  for i in args:
    result = result +i
  return result

add_many(1,2,3,4,5)
```

입력이 몇개가 될지 모를때는 매개변수에 \*를 앞에 붙여줍니다. 그러면 사용을 할때 인자의 개수를 여러개를 넣을 수 있습니다.

```
#함수의 결과값은 언제나 함수이다.

def add_mul(a,b):
  return a+b, a*b 
  return a*b

result =add_mul(3,4)
print(result)
```

함수의 결과값은 언제나 한개입니다. 그런데 왜 첫번째 return엔 식이 2개고 그 밑엔 2번째 return이 있을까요? 먼저 return에 식이 2개면 두 결과값이 (7,12)로 하나의 튜플값으로 나오게 됩니다. 그리고 함수는 리턴값을 만나면 break처럼 함수를 종료하기 때문에 첫번째 그 밑으로의 return은 의미가 없죠.. 만약 조건문이 따로없다면..ㅎ

```
result1, result2 = add_mul(3,4)

print(result1)
print(result2)
```

같은 함수에 저렇게 2개의 변수로 받으면 한쌍으로 나온 튜플값이 아니라 그 안에 값 1개씩을 순서대로 가져갑니다!

```
#함수에서 선언한 변수의 효력 범위...

a =1
def vartest(a):
  a =a+1

vartest(a)
print(a)
```

매개변수는 그 함수에만 쓰이는 다른 녀석입니다. 자바에서도 같은 원리죠?

```
a=1
def vartest():
  global a
  a = a+1
  return a
print(vartest())
```

따라서 저렇게 global을 앞에붙여주면 마치 자바에서 this. 를 한것처럼 사용할 수 있습니다.

**lambda**

```
add = lambda a,b : a+b
r = add(3,4)
print(r)
```

람다는 함수를 더 간단히 만든것입니다. add라는 함수를 만들 때 람다를 쓰고 매개변수와 변수를 이용한 식만 적으면 끝.. 엄청 간단하네요ㅎㅎ

**사용자 입력**

사용자 입력은 input()메서드를 사용합니다. 그리고 이 input()메서드는 str, 즉 문자열 값만 반환하기 때문에 숫자를 반환하려면 int(input())으로 해주어야 합니다!

```
def num(a):
  
  if a%2==1:
   print("홀수입니다.")

  else:
   print('짝수입니다')  

num(int(input()))
```

값을 넣으면 홀수인지 짝수인지 판별해주는 함수를 만들었습니다.

```
def add_mul(choice, *args):
  if choice =="add":
    result =0
    for i in args:
      result +=i
    
  elif choice=="mul":
    result=1
    for i in args:
      result *=i
  return result

print(add_mul('mul',1,2,3,5))
```

매개변수를 2종류로 두었습니다. choice에 들어온 값을 판단하여 다른 로직을 수행하도록 하였습니다!

**파일입출력**

자바에서와 같이 파일입출력인데 파이썬에서는 상상이상으로 간단합니다.

```
# 파일 생성

f = open("파일이름.txt",'w')
f.close()
```

이게 끝.. ㅋㅋㅋ 방금 하나 생성했습니다.

```
f = open("새파일.txt", 'w')
for i in range(1,11):
  data = "%d번째 줄입니다.\n" %i
  f.write(data)
f.close()
```

또 write()함수로 그 안에 내용을 작성할 수 있습니다. 그리고 항상 파일을 열면 close()로 닫는것까지 해주어야 합니다.

```
f = open("새파일.txt", 'r')

while True:
  line = f.readline()
  if not line:
    break
  print(line)

f.close()
```

'r'은 있는 파일을 읽을 때 사용합니다. line이라는 변수를 만들고 한 라인씩 담아옵니다. 그리고 더이상 line이 없으면 break로 빠져나옵니다.

```
#read()

f = open("새파일.txt",'r')
lines = f.read()

print(lines)
f.close()
```

여기는 그냥 read()를 썼네요. read()는 전체 내용을 하나의 문자열로 처리해줍니다!

```
f = open('새파일.txt' ,'a')
for i in range(11,21):
  data = " %d번째 줄입니다.\n" %i
  f.write(data)

f.close()
```

'a'는 수정입니다. 원래 있던 데이터에 추가를 합니다 아마 append의 a가 아닐까..함수는 똑같이 write()를 씁니다.

```
with open("새파일.txt", 'w') as f:

  f.write("No pain No gain")
```

with는 자동으로 닫아줍니다. 더 간단하게 파일입출력을 할 수가 있죠

**클래스살짝!**

```
class Calculator:
  def __init__(self):  #생성자
    self.result=0
  

  def add(self,num):
    self.result +=num
    return self.result

cal1 = Calculator()

print(cal1.add(3))
print(cal1.add(3))
print(cal1.add(3))
```

Calculator라는 클래스를 만들었는데요 def로 \_\_init\_\_(self)를 만들었습니다. 생성자인데요 자세한 설명은 다음에 하겠습니다. ㅎㅎ 로직은 이 클래스가 갖고있는 함수중 add()를 사용하여 매개변수에 들어간 인자를 num에 넣어줍니다. 그리고 self의 result에 넣어준답니다!

이번시간엔 여기까지 하겠습니다!