---
title: "[Java] Set Iterator Hashmap Calendar"
date: 2023-11-13 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
---


이번 시간에는 Set이라는 자료구조와 iterator와 HasMap, 그리고 향상된 for문인 forEach를 알아보겠습니다.

**Set : 집합**

**\- HashSet**

집합에서는 중복되는 원소를 포함할 수 없습니다. 그리고 특징은 저장된 값들은 순서가 없다는 것인데요, 따라서 값의 유무 검사에 특화되어 있는 자료구조입니다. HashSet은 해시 코드로 유무 검사가 진행되고 속도가 상대적으로 좋습니다.

**\-문법**

HashSet<E> 객체명 = new HashSet<>();

```
public static void main(String[] args) {
		HashSet<String> bloodTypeSet = new HashSet<>();
		bloodTypeSet.add("A");
		bloodTypeSet.add("B");
		bloodTypeSet.add("O");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		bloodTypeSet.add("AB");
		
		System.out.println(bloodTypeSet);
```

메인메소드를 가져왔습니다. HashSet은 Set자료구조를 상속받은 클래스이기 때문에 중복된 자료가 없다고 했죠? 따라서 add()메소드로 AB라는 문자열값을 여러번 넣어도 하나의 AB로 취급됩니다. 실행시켜보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/1.png)

오잇? ㅋㅋ 저 형식을 보니 toString메소드를 재정의 했나보군요..

그런데 여기서 질문 HashSet은 인덱스가 없는데 어떻게 자료를 가져올 수 있을까요?그것은 바로 재정의된 toString메소드에 있는 iterator때문입니다. 그럼 .toString()을 붙인 뒤 원본 소스로 들어가보겠습니다.

```
public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }
```

저번에 한번 봤었죠? 이건 컬렉션이라는 이름의 자료구조 인터페이스 소스코드에 있는 코드입니다. 보시면 Iterator<E> it = iterator();라는게 보이시죠?그리고 사실 iterator 앞에 this. 가 생략되어있습니다. 따라서 this.iterator라는 거죠.

iterator를 봐봅시다.

**\- Iterator**

순서가 있는 객체.

\*iterator()

순서가 없는 객체에 순서를 부여하거나, 순서가 있어도 iterator방식의 순서로 변경하고자 할 때 사용합니다. 따라서 인덱스가 없는 Set에서 인덱스를 부여하고 따라서 출력까지 할 수 있었던 거죠.

hasNext()를 통해 다음 값이 있는 지 검사하고, next()를 사용하여 값을 가져옵니다.

따라서,

```
Iterator<String> iter = bloodTypeSet.iterator();
```

이런 식으로 bloodTypeSet.iterator()로 이터레이터 타입으로 만들어 줄 수 있습니다. 뭔가 비슷하지 않나요? 맞습니다!

아까 toString에서 본 Iterator<E> it = iterator(); 이것이죠. 생략되어 있는 this.를 붙이면 Iterator<E> it = this.iterator(); 죠 이제 객체 iter를 사용해서 순서가 생긴 Set자료를 사용할 수 있습니다.

```
while(iter.hasNext()) {
			System.out.println(iter.next());
		}
```

이런식으로 while문으로 객체 iter의 원소들을 전부 출력할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/2.png)

잘 출력이 되네요 ^^

**Map**

**\- HashMap**

Key와 Value 한 쌍으로 저장되며, 검색의 목적을 가지고 있습니다. Key는 중복된 값을 넣으면 Value가 최근 값으로 수정되고,중복되지 않는 값을 넣으면 새롭게 한 쌍이 추가 됩니다. Key는 중복 없이 들어가며 Value는 중복이 허용됩니다. 따라서 key값은 중복이 되지 않으니 아까 배웠던 set타입입니다.

**\-문법**

HashMap<K, V> 객체명 = new HashMap<>();

K에는 키값의 타입, V에는 밸류값의 타입을 적으면 됩니다.

바로 만들어보죠.

```
HashMap<String,Integer> fruitMap= new HashMap<>();
		fruitMap.put("사과", 3000);
		fruitMap.put("배", 4000);
		fruitMap.put("수박", 8000);
		fruitMap.put("자두", 800);
		System.out.println(fruitMap.size()); // 한쌍이다
		System.out.println(fruitMap);
		
		System.out.println(fruitMap.get("배"));//키 값을 주면 밸류를 준다.
```

사과 - 3000

배 - 4000

수박 -8000

자두 -800

으로 쌍을 정했죠? 과일종류에는 중복이 없어야 합니다. 수박이 2개 있으면 안되겠죠? 다만 가격은 다른 과일이라도 같을 수 있으니까 중복이 될 수 있고요.. 편리합니다. 여기서 fruitMap.size()와 fruitMap을 출력하면 어떻게 될까요? 또 fruitMap.get()으로 키 값을 가져와서 출력한다면 어떤 일이 일어날까요? 봅시다!

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/3.png)

size()는 8이 아니라 4가나오네요. 한 쌍을 1로 취급하나봅니다. 그리고 객체명을 입력했을 때에는 두번 째줄처럼 이쁘게 나오네요. 역시 toString()을 재정의 했겠죠? 그리고 "배"라는 키값을 출력했더니 배가 안 나오고 그의 짝인 밸류 4000원이 나왔습니다! 재밌네요 ㅎㅎ

이제 키와 밸류를 분리해봅시다. keyset()과 values()를 이용하겠습니다.

먼저 아까 배운 iterator를 사용하겠습니다.

```
Iterator<String> fruitName = fruitMap.keySet().iterator();
```

key값은 과일의 이름이었으니까 fruitName이라는 이름으로 iterator타입의 객체를 만들었습니다. keySet()으로 key를 분리해주고 iterator로 인덱스를 부여했습니다.

```
while(fruitName.hasNext()) {
			System.out.println(fruitName.next());
		}
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/4.png)

잘 출력이 되네요 ^^

이제 밸류값을 분리해볼건데요. 그 전에 하나 더 배울 게 있습니다.

**forEach(빠른 for문, 향상된 for문)**

바로 forEach라는 향상된 for문을 배워보겠습니다. 보통 for문은 선언식 ; 조건식 ;증감식으로 조건문이 만들어지는데 향상된 for문은 세미콜론이 아닌 그냥 콜론으로 조건문이 이어집니다. 반복횟수도 그에 맞춰 알아서 이루어지죠!

**문법**

for(자료형 변수명 : iterator){

반복할 문장;

}

한번 연습을 해보겠습니다.

```
public static void main(String[] args) {
		String[] arData  = {"안녕","하세요"};
		for(String data : arData) {
			System.out.println(data);
		}
	}
```

String 타입의 배열 arData를 선언하고 초기화까지 했습니다. 문자열 "안녕", "하세요" 2개가 들어있습니다. 이제 for( String data : arData)를 해주면 이 배열에 접근해서 data에 대입됩니다. 그리고 바디가 실행되고 그 다음 반복엔 data에 arData의 다음 값이 data에 들어가고 바디가 실행됩니다. 언제까지? arData의 길이만큼이요~ 알아서 반복횟수를 정해주니 편합니다 ^^ 2차원 배열은 어떻게 할까요?

```
public static void main(String[] args) {
		int [][] arrData = {
				{10,25,30},
				{11,24,35},
				{12,23,40},
				{13,22,45},
				{14,21,50},
		};

		//학생별 총점과 평균을 출력한다.
		//빠른 for문만 사용한다.
		
		int total = 0;
		double avg= 0.0;
		int num=0;
		for(int[] arData :arrData ) {
			num++;
			total=0;
			for(int data : arData) {
				
				total +=data;
			}
			avg = (double)total/arData.length;
			avg= Double.parseDouble(String.format("%.2f", avg));
			System.out.println(num + "번 학생 총 점: " +total + "점");
			System.out.println(num + "번 학생 평균: " +avg + "점");
		}
		
	}
```

연습문제입니다. 간단하게 설명드리면 2차원 배열은 한번 접근하면 행에 접근합니다. 그 5개의 행 하나씩 또 쪼개져서 3개의 값으로 이루어졌습니다.

그래서 이중으로 향상된 for문을 적어주면 됩니다. 처음에 int\[\]arData를 선언하고 arrData를 적으면 2차원 배열에 한번 접근하고 그 것을 arData에 넘겨줍니다. 여기엔 행단위로 값이 들어가있죠. 이제 다시 다음 for문으로 int data로 arData를 받습니다. 이제 그 행 안에 있는 열값들이 들어오죠. 이 문제는 학생들의 3과목 점수를 입력하여 각 학생의 총점과 평균을 구하는 작업입니다. 나머지는 크게 어려운 문제가 없죠.

이제 다시 HashMap으로 돌아가서 밸류값들을 분류해보겠습니다.

```
//과일의 평균 가격 출력
int sum=0;	
double avg=0.0;
for(int data : fruitMap.values()) {
			sum += data;
		}
		avg = (double)sum/fruitMap.values().size();
		avg= Double.parseDouble(String.format("%.2f", avg));
		System.out.println(avg);
	}
```

과일가격(value)의 평균을 구하는 문제입니다. int sum과 double avg를 하여 선언 및 초기화를 해줍니다. 그리고 향상된 for문으로 fruitMap.values()로 하나씩 data에 반복 대입하고 그것을 sum에 누적중첩시킵니다. 그리고 avg에는 그 총합이 담긴 sum을 values의 사이즈(갯수)만큼 나눠주죠. avg= Double.parseDouble(String.format("%.2f", avg)); 이것은 포맷 메소드로 C언어에서 했던 것처럼 소수점 2자리수까지 나타내는 것입니다. 이제 avg를 출력하면


![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/5.png)
잘 나오네요 ^^ 재밌습니다!

**Date**

Date는 말 그대로 시간과 관련된 코드를 갖고있는 클래스입니다.

```
public class DateTest {
	public static void main(String[] args) {
		Date today = new Date();
		System.out.println(today);
	}
```

Date 타입의 객체 today를 만들었습니다. 그리고 이 객체를 출력하면?

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/6.png)

이렇게 현재 시간이 나옵니다 ㅎㅎ 그런데 형식이 좀 가독성이 떨어지는 것을 볼 수 있습니다.

이럴때 필요한게 SimpleDateFormat 클래스입니다.

```
public static void main(String[] args) {
		Date today = new Date();
		System.out.println(today);
		
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy년 MM월 dd일");
		System.out.println(sdf.format(today));
	}
```

2줄이 더 생겼습니다. 자바 라이브러리에 있어서 import해야합니다. SimpleDateFormat 타입의 객체 sdf를 만들고 생성자엔 "yyyy년 MM월 dd일" 로 적어주고 sdf.format이라는 메소드에 아까 만들었던 Date타입의 객체 today를 출력하면

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/7.png)

이렇게 우리가 원하는 형식으로 만들 수 있습니다. 형식에 관한것은 SimpleDateFormat의 링크를 타고 들어가면 여러가지를 볼 수 있습니다.

**Calendar**

이번엔 캘린더의 클래스를 알아보겠습니다. 캘린더는 Date와 다르게 추상클래스입니다. 따라서 new연산자를 이용하지 않죠. 왜냐하면 달력은 각자 인스턴스화를 해서 각각 만들면 안 되고 하나의 달력과 시간만이 존재해야하기 때문입니다. 그럼 new연산자가 없을테니 생성자도 없을테고 객체 초기화를 어떻게 할까요?

```
        Calendar cal = Calendar.getInstance();
		cal.set(2021, 11,04);
		System.out.println(cal);
```

cal이라는 Calendar 객체를 만들고 new Calendar가 아닌 Calendar.getInstance()라는 메소드로 만듭니다.

그리고 set이라는 메소드로 우리가 원하는 시간을 넣어주고 이 객체를 출력하면

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/8.png)

엄청 긴게 나오죠? 사실 저것보다 몇배는 더 긴 출력물이 나옵니다. 보시면 중간에 우리가 작성한 시간이 들어있기는 합니다. 보기가 너무 불편하죠.

이제 우리 아까 배운 SimpleDateFormat클래스를 사용합니다.

```
public static void main(String[] args) { //캘린더는  추상클래스라 new 연산자 사용 안 함.
		Calendar cal = Calendar.getInstance();
		cal.set(2021, 11,04);
		System.out.println(cal);
		
		SimpleDateFormat sdf= new SimpleDateFormat("yyyy-MM-dd");
		System.out.println(sdf.format(cal.getTime()));//Date타입으로 받아야해서 Calendar를 형변환
		//월 입력은 0부터 시작 따라서 12월이면 11입력해야함 12하면 1년추가하고 1월달임
		
		//Month를 가져올 때에도 0부터 시작해서 +1해줘야함.
		System.out.println(cal.get(Calendar.MONTH)+1);
	}
```

SimpleDateFormat sdf= new SimpleDateFormat("yyyy-MM-dd");로 형식은 조금 다르지만 sdf객체에 "yyyy-MM-dd"식으로 초기화를합니다. 그리고 sdf.format() 메소드 안에 바로 cal를 넣지 않았습니다. 왜냐면 cal은 Calendar클래스고 format()메소드엔 Date 클래스만 들어갈수 있거든요. 그래서 cal.getTime()를 넣어서 cal를 Date타입으로 바꿔주고 넣었습니다. 중요한 것은 우리가 12월을 출력하고 싶으면 월 칸에 11을 입력해야합니다. 왜냐하면 월은 0부터 시작해서 0이 1월 , 11이 12월 이런식으로 진행됩니다. 그래서 12월 4일을 출력하기 위해선 set()메소드에 11월 4일을 입력해야하죠.

마찬가지로 월만 출력하고 싶을때도 +1로 합니다.

이번시간에는 여기까지 하겠습니다~ 감사합니다