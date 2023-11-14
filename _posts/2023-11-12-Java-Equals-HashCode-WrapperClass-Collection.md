---
title: equals 재정의 hashCode Wrapper클래스 Collection
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


이번 시간에는 Object클래스에서 equals()메소드 재정의와 Wrapper Class로 박싱,언박싱을 알아보고 Collection Framework에서 list부분 조금 짚어보겠습니다.

먼저 저번 시간에 못했던 equals() 메소드 재정의를 해볼게요.

```
package day15;

public class School {
	public static void main(String[] args) {
		
		//결과가 true가 나올 수 있게 equals를 재정의
		Student shin = new Student(1,"신동희");
		System.out.println(shin.equals(new Student(1,"신동희")));
	}
}
```

문제는 School이란 클래스에서 Student 타입의 객체 shin에 생성자를 만들고

System.out.println(shin.equals(new Student(1,"신동희")));

이라는 문장을 넣었습니다. equals 메소드에서는 true를 리턴할까요 false를 리턴할까요? 당연히 false입니다.

왜냐하면 new연산자가 붙었기 때문이죠. 따라서 shin객체와 다른 새로운 주소값을 만들어냈기 때문에 false를 리턴합니다. 이제 여기서 이 equals에서 true를 리턴하기 위해서 equals메소드를 재정의해보도록 하겠습니다.

```
@Override
	public boolean equals(Object obj) {
		if(this == obj) {return true;}
		
		if(obj instanceof Student) {
			Student std = (Student)obj;
			if(this.num == std.num) {
				return true;
			}
		}
		return false;
		
	}
```

Student클래스에서 재정의해보았습니다. 일단 매개변수를 Object클래스 obj로 해서 모든 값이 일단 들어올 수 있게 해두었습니다. 그 후 equals메소드를 사용하는객체와 obj가 주소값이 같으면( ==은 주소값 비교입니다) 뭐 당연히 true겠지요. 하지만 아까 처럼 new연산자로 주소값이 다르지만 그 내용이 같을 때 true를 리턴하기 위해선 먼저 obj가 Student 타입일때(타입비교), 그리고 사용한 객체의 num과 Student타입으로 다운캐스팅된 obj의 num이 같다면 같은 학생번호를 가지므로 같은 학생입니다. 따라서 true를 리턴하죠. 이 중 하나라도 해당이 안 된다면 false를 출력하도록 했습니다. 이제 true가 출력될 수 있겠죠?

**hashCode()**

hashCode는 간단하게 말해서 주소값이라고 보면 됩니다. 자세한 건 나중에 또 배우게 되면 포스팅하도록 하겠습니다.

hashCode 메소드는 Object에 있는 메소드 중 하나입니다. 바로 예시로 가볼게요.

```
package day15;

import java.util.Random;

public class HashCodeTest {
	public static void main(String[] args) {
		String data1 = "ABC";
		String data2 = new String("ABC");
		
		Random r1 = new Random();
		Random r2 = new Random();
		
		System.out.println(r1.hashCode());
		System.out.println(r2.hashCode());
		System.out.println(data1.hashCode());
		System.out.println(data2.hashCode());
	}
}
```

data1에는 "ABC"로 값 위주의 문자열을 선언하고, data2에는 new String("ABC")로 주소값 위주의 문자열을 선언하였습니다. 그리고 Random 클래스로 객체 r1,r2를 선언하고 각각 생성자로 주소값을 대입하였습니다. 이제 프린트문으로 저 4개를 hashCode메소드로 출력해볼게요.

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/1.png)

대표사진 삭제

사진 설명을 입력하세요.

헉 ㅋㅋ 처음 Random클래스 두 객체는 역시 다른 해시코드를 출력했지만 String 클래스의 data1,data2는 같은 해시코드를 출력했습니다. 왜그럴까요? 우리가 문자열 "ABC"를 선언할 때 RAM의 한 부분 constant pool (c.p)라는 곳에 "ABC"의 주소값이 생성됩니다. 이것은 객체가 각각 다른 주소값을 가진 것과 별개로 하나의 해시코드를 갖고 있다고 합니다. 그래서 같은 해시코드를 출력한 것이죠. 더 깊게 파고들면 끝도 없어서 나중에 또 배우겠습니다. 물론 포스팅과 함께요 ^^..

**Wrapper Class : 기본 자료형들의 클래스 타입**

이게 어떤 것이냐면, 우리가 흔히 사용하는 int, double, char .. 등등 이런 기본 자료형들보다 더 큰 범위를 갖고있는 하나의 클래스타입입니다.

예를 들어, int data; 를 선언하고 객체처럼 data.~ 이런식으로 사용하지 못합니다. 하지만 Integer data\_I = new Integer(data\_i);로 선언한다면 마치 하나의 주소값을 갖고 있는 객체가 되는거죠. 이때는 data\_I.~이런식으로 Integer클래스에 있는 다양한 메소드를 사용할 수 있습니다.

\-문법

클래스 타입 객체 = new 클래스 타입(일반 타입); //boxing

클래스 타입의 객체는 풀스펠링으과 첫문자를 대문자로 쓰면 됩니다.

ex) Integer data\_I = new Integer(data\_i);

일반 타입 변수 = 객체.000Value(); //unboxing

ex) data\_i = data\_I.intValue();

JDK4 버전 이상부터는 auto를 지원합니다. auto는 저렇게 일일이 쓰는게 너무 번거로워서 간단하게 하는 겁니다.

클래스 타입 객체 = 일반 타입; //auto boxing

ex) Integer data\_I = data\_i;

엄청 간단해졌죠?

일반 타입 변수 = 객체; //auto unboxting

ex) data\_i = data\_I;

Wrapper 클래스를 사용하는 이유

원시타입(일반타입)을 박싱하면 다양한 메소드를 제공받을 수 있으며, Casting도 자유로워질 수 있습니다. 반드시 객체로 사용해야 할 때에도Wrapper 클래스로 변경하여 사용할 수 있습니다.

**컬렉션 프레임워크(Collection Framework) : 자료구조**

많은 데이터를 쉽고 효과적으로 관리할 수있는 표준화된 방법을 제공하는 클래스들의 집합입니다. 의미 없는 여러개의 데이터들이 자료구조를 거쳐 하나의 가치가 되는 정보가 되는 것이죠. 그 거름망을 자료구조라고 생각하시면 됩니다. 그리고 이 컬렉션 프레임워크에는 여러가지 클래스가 있는데 오늘은 List를 살펴보겠습니다.

**1\. List extends Collection**

\-구현 클래스

**Vector** : 보안성 강화, 처리량 감소, 용량 관리, 하지만 정말 옛날에 만든거고 요즘엔 거의 쓰이지 않는다고 합니다. 예전에는 데이터보안의 일환으로 한 개씩 처리해서 넘어가는 알고리즘을 채택했다고 하네요.

**LinkedLis**t : FILO로 인해 넣을 때는 빨라도 뺄 때에는 느리다. 하나의 필드에는 자신의 값과 별개로 다음 필드의 주소값을 갖고있는데요 따라서 마지막 필드에 접근하기 위해선 그 전의 모든 필드를 거쳐야하죠. 즉 FILO(First In Last Out)의 방식으로 처리됩니다. 마치 Stack과도 같죠.

**\*ArrayList** : 인덱스로 데이터를 관리하기 때문에 값을 가져오**는** 속도가 빠르다. 대부분의 실무에서 쓰이는 클래스입니다. 이부분을 자세히 보도록 하죠.

**ArrayList**

컬렉션 클래스 중 실무에서 가장 많이 사용되는 클래스입니다.

배열의 특징인 인덱스를 이용하여 값을 저장하고 관리합니다.

배열과 ArrayList의 차이

배열은 길이에 제한을 두어야 할 때 자주 사용되고,

ArrayList는 몇 개의 데이터가 들어올 지 알 수 없을 때 사용합니다. 배열같은 경우 우리가 처음 설정한 length만큼 고정이되어서 초과한 인덱스에 값을 넣어야할 때는 번거롭게 그 데이터들을 복사하고 새로운 배열을 선언한 뒤 추가해야하지만, ArrayList는 add라는 메소드로 자동으로 인덱스가 추가되고 값을 삽입합니다. 엄청 편하죠? 그래서 많이 쓰이는 것 같습니다.

문법

ArrayList<E> datas = new ArrayList<>();

잉? 전에 보지 못했던 <E>가 있네요? 이 꺽새안에 E는 제네릭이라고 합니다. 제네릭은 '이름이 없다'라는 뜻인데요, 타입을 지정하지 않는 기법이라 사용할 때 우리가 원하는 타입을 지정하도록 합니다. 여기에는 아까 boxing때 배웠던 클래스타입의 자료형을 써주면 됩니다. 예를 들어,

ArrayList<Integer> datas = new ArrayList<>(); 이런 식으로요.

```
ArrayList<Integer> datas = new ArrayList<>();
		datas.add(10);
        System.out.println(datas.size());
		
		System.out.println(datas);
```

정수타입 배열을 선언했지만 크기는 정하지 않았습니다. 그리고 add()메소드로 10을 추가했습니다.

그 다음 그 배열의 칸의 개수를 알려주는 size()메소드와 그냥 객체 datas를 출력했습니다. 결과는 어떻게 나올까요?

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/2.png)

대표사진 삭제

사진 설명을 입력하세요.

size는 1 이나왔습니다. 자동으로 추가만 했을뿐인데 배열의 인덱스도 추가가 된것이죠. 그런데 신기한건 datas객체를 출력하면 주소값이 나와야 하는데 왜 \[10\]이라는 것이 출력되었을까요? 그럼 저번 시간에 배운 것처럼 ArrayList클래스에서 toString을 재정의 한것이죠. 이제 datas.toString()으로 하고 자세히 뜯어봅시다.

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

이것은 제가 toString()의 링크를 타고가서 그 소스코드를 가져온 것입니다. 역시 재정의 되어있었군요. 간단하게 말하자면 iterator로 순서를 만들어준 다음 next()로 \[ 를 시작으로 하나씩 값을 넣어 준뒤 더이상 그 다음 값이 없으면 \]를 리턴하고 메소드를 빠져나오네요. 따라서 우리가 datas라는 객체만 출력을 해도 toString에서 저런식으로 출력을 해주네요. 그럼 add를 여러번하면?

```
ArrayList<Integer> datas = new ArrayList<>();
		datas.add(10);
        datas.add(20);
        datas.add(30);
        System.out.println(datas.size());
		
		System.out.println(datas);
```

실행해보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/3.png)

대표사진 삭제

사진 설명을 입력하세요.

신기하죠? 자동으로 인덱스가 추가되고 그 자리에 값이 들어왔습니다.

정말 JAVA는 알수록 재미있는 언어인것 같습니다. ^^