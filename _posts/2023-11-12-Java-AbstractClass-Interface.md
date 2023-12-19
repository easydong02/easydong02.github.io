---
title: "[Java] 추상클래스 인터페이스"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

이번 시간에는 추상클래스와 인터페이스에 대해서 알아보겠습니다!

**추상 클래스**

필드 안에 구현이 안 된 메소드가 선언되어 있는 클래스를 고추상 클래스라 합니다. 이 때 구현되지 않은 메소드를 추상 메소드라고 부릅니다.

따라서 반드시 재정의를 통해구현을 해야지만 메모리에 할당되기 때문에 강제성을 부여하기 위해서 추상 메소드로 선언합니다. 강제성을 부여하지 않으면 코드가 적혀있지 않은 메소드를 확인하지 못하고 다른 클래스에서 그 메소드를 그냥 차용할 수 있는 문제를 방지하기 위한 것입니다.

**추상 클래스 선언**

abstract class 클래스명{

abstract 리턴 타입 메소드명 (자료형 매개변수1,...);

\*\*일반 메소드도 선언이 가능합니다. 왜냐하면 추상클래스도 일단 클래스이기 때문입니다.

}

이제 한번 추상 클래스를 만들어보겠습니다. 먼저 가전제품이라는 부모 추상클래스를 만들고 일반 자식 클래스들을 만들게요.

```
package day12;

public abstract class Electronics {
	public abstract void on();
	public abstract void off();
	public final void printManual() {
		System.out.println("전자제품입니다.");
	}
}
```

class 앞에 abstract를 붙여서 추상 클래스를 만들었습니다. 필드엔 on, off , printManual 이라는 메소드를 만들었습니다. on,off 메소드는 중괄호가 없습니다. 따라서 구현이 안 된 메소드입니다. 원래라면 오류가 나지만 이 메소드도 마찬가지로 앞에 abstract 를 붙여서 추상메소드로 만들었습니다. 그리고 printManual메소드는 추상이 아닌 상수로 만들고 안에 print문을 넣었습니다. 이제 다른 클래스를 만들어보겠습니다.

```
package day12;

public class Refrigerator extends Electronics{

	@Override
	public void on() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void off() {
		// TODO Auto-generated method stub
		
	}

}
```

처음만든 Electronics클래스를 상속받은 Refrigerator 클래스를 만들었습니다. 부모 클래스의 on, off 메소드는 구현이 안 되어있죠? 그래서 그냥 상속만 한다면 이런 에러문구가 뜹니다. 따라서 위와 같이 오버라이딩으로 자식클래스에서 하나하나 구현을 따로 했습니다.

![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/1.png)

**The type Refrigerator must implement the inherited abstract method Electronics.on()**

해석하자면 냉장고 타입은 무조건 추상클래스인 부모클래스에서 상속만 받지말고 구현까지 하라는 것이죠. 그래서 재정의(오버라이딩)을 통해서 자식 클래스에서 직접 구현을 해야합니다. 그렇다면 이 추상메소드를 상속받은 모든 자식 클래스에서는 on off 메소드를 다 직접 구현해야되는데 귀찮게 왜 이러는거죠? 이유는 자식클래스에서 각각 다른 형태로 필요한 메소드이기 때문입니다. 예를 들어, 같은 가전제품이라곤 하지만 냉장고, 청소기, 세탁기 등등.. 다 전원을 키는 방법이랑 끄는 방법이 다르겠죠? 근데 이것은 부모클래스에서 하나로 정해버리고 그냥 상속을 받아버리면 곤란합니다. 그래서 강제적으로 자식클래스에서 그 클래스에 맞게 구현을 하라는 의미죠.

**인터페이스(Interface) : 틀**

추상 클래스를 고도화 시킨 문법입니다. 여기에는 상수와 추상메소드만 존재하는데요, 구현은 인터페이스를 지정한 클래스에서 진행하고,

인터페이스를 다른 클래스에 지정할 때에는 implements 키워드를 사용합니다. 아까 말했듯이 추상클래스는 그 필드안의 추상메소드 등으로 자식클래스에서 구현을 강요하는데요, 이것을 전문적으로 하는 것이라고 보면 됩니다. 추상클래스와 달리 여기에는 일반메소드가 아예 들어오지 못합니다. 따라서 말 그대로 '틀' 만 존재하는 것입니다.

이제 동물이라는 인터페이스를 만들고 그것을 지정하는 클래스들을 만들어서 보여드릴게요

```
package inter;

public interface Animal {
	final static int eyes =2; //공유해야 하기때문에 final과 static은 생략이 된다.
	int legs =4;
	
	void eat();
	void sleep();
	void getHand();
	void shakeTail();
}
```

먼저 동물이라는 틀을 만들고 싶었습니다. 여기에는 상수와 추상메소드 밖에 없다고 했죠? 그래서 그냥 int legs =4; 이렇게 해도 앞에 상수 final과 static이 붙습니다. 왜냐하면 다른 클래스들과 공유해야하기 때문이죠. 메소드도 마찬가지입니다. 그래서 eat, sleep, getHand, shakeTail 메소드를 만들어 봤습니다. 이제 다른 일반 클래스들을 만들어볼게요

```
package inter;

public class Cat implements Animal{

	@Override
	public void eat() {
		System.out.println("참치를 먹는다.");
		
	}

	@Override
	public void sleep() {
		System.out.println("잠을 잘 잔다.");
		
	}

	@Override
	public void getHand() {
		System.out.println("손을 주지 않는다.");
		
	}

	@Override
	public void shakeTail() {
		System.out.println("꼬리를 흔들지 않는다.");
		
	}

}
```

고양이라는 클래스를 만들었습니다. 인터페이스를 지정할때는 상속과 달리 implements를 씁니다. 그리고 인터페이스에 있는 모든 메소드는 추상메소드이기 때문에 고양이 클래스에서 오버라이딩을 통해 모두 재정의해주는 모습이 보입니다. 자 그럼 이제 다른 클래스를 만들어 볼게요 저는 이제 호랑이를 만들고 싶습니다.

```
package inter;

public class Tiger implements Animal{

@Override
	public void eat() {
		System.out.println("참치를 먹는다.");
		
	}

	@Override
	public void sleep() {
		System.out.println("잠을 잘 잔다.");
		
	}

	@Override
	public void getHand() {
		System.out.println("손을 주지 않는다.");
		
	}

	@Override
	public void shakeTail() {
		System.out.println("꼬리를 흔들지 않는다.");
		
	}
```

고양이 클래스와 똑같은 바디를 넣었습니다. 근데 호랑이 클래스에서는 getHand와 shakeTail메소드는 쓰기가 싫어지군요. 그런데 그것을 안 쓸 수 없습니다. 왜냐하면 구현이 안된 추상메소드라 무조건 다 구현을 해야하거든요.. 여기선 필요한게 Adapter입니다.

**추상클래스와 인터페이스 간의 관계**

인터페이스를 클래스에 바로 지정하면 모든 메소드에 강제성이 부여되어서 전부 다 구현해야 합니다. 하지만 일반적인 상황에서는 모든 것이 아닌,

필요한 메소드를 골라서 재정의 해야 하겠죠? 인터페이스를 직접 지정하지 않고 다른 클래스에 지정한 후 바디를 만들어 놓는다면, 강제성이 소멸되고 이 클래스를 상속받아서 필드를 구현한다면, 골라서 재정의가 가능합니다. 이 때 중간에서 강제성을 없애주는 클래스를 추상 클래스로 선언하며, 클래스 이름 뒤에 Adapter를 붙여서 목적을 알려줍니다.

이제 Adapter를 만들어서 호랑이 클래스에서 제가 필요한 부분만 사용해볼게요.

```
package inter;

public abstract class AnimalAdapter implements Animal{
 @Override
public void eat() {
	// TODO Auto-generated method stub
	
}
 @Override
	public void sleep() {
		// TODO Auto-generated method stub
		
	}
@Override
	public void getHand() {
		// TODO Auto-generated method stub
		
	}
@Override
	public void shakeTail() {
		// TODO Auto-generated method stub
		
	}
}
```

AnimalAdapter라는 추상클래스를 만들었습니다. 이 추상 클래스는 Animal인터페이스를 지정합니다. 그리고 모든 메소드를 넘겨받았습니다.

그리고 아까 만들었던 Tiger클래스를 수정해볼게요.

```
public class Tiger  extends AnimalAdapter{

	@Override
	public void eat() {
		// TODO Auto-generated method stub
		
	}

	@Override
	public void sleep() {
		// TODO Auto-generated method stub
		
	}

	
}
```

자 만들었습니다. Animal이 아닌 어댑터를 상속받으면서 제가 원하는 메소드만 골라서 재정의할 수 가 있었습니다. 재밌죠?ㅎㅎ

**마커 인터페이스(Marker Interface)**

클래스들을 그룹화하기 위한 목적으로 사용합니다. 인터페이스는 지정한 클래스의 부모이며, 모든 자식은 부모의 타입이므로 마커 인터페이스를 지정받은 클래스들이 하나의 타입으로 묶이게 된다.

예를 들어, 넷플릭스라는 프로그램을 만든다고 한다면 하나의 작품이 하나의 종류로만 묶이게 될까요? 여러가지 종류가 다 들어가 있겠죠.. 애니메이션이면서 영화거나 애니메이션이면서 시즌제 이거나.. 스릴러면서 드라마거나 애니메이션이거나.. 한번 만들어보겠습니다.

```
package maker;

public class Video {

}
```

Video라는 클래스를 만들고

```
package maker;

public class Onepiece extends Video{

}
```

video클래스를 상속받는 원피스 클래스를 만들었습니다. 자그럼 원피스는 비디오에도 속하지만 애니메이션에도 속하죠? 그런데 다중 상속은 JAVA에서 '원칙'적으로 지원하지 않습니다. 따라서 이 때 제각각의 클래스들을 묶어줄 수 있는 마커 인터페이스가 필요합니다.

```
package maker;

public interface Animation {;}
```

Animation이라는 마커 인터페이스를 만들고 안에는 아무것도 작성하지 않았습니다.

그럼 이제 원피스 클래스로 돌아가서 수정을 해볼게요.

```
package maker;

public class Onepiece extends Video implements Animation{

}
```

이제 원피스는 video의 상속을 받으면서 인터페이스를 지정도 하여 Animation이라는 타입이 하나 추가 되면서 효율적으로 여러가지 클래스들을 관리하기 쉬워졌습니다! 정말 재밌네요 ㅎㅎ

**다중 상속**

여러 부모 클래스를 상속하는 것을 다중 상속이라고 합니다.JAVA는 모호성때문에 다중 상속을 지원하지 않습니다.하지만 JDK8버전 부터는 인터페이스에서 메소드를 구현할 수 있도록 허용하였습니다. 이를 default메소드라고 하며, 여러 개를 지정할 수 있는 인터페이스 특성 상 다중 상속을 지원하는 것이나 다름이 없습니다.

extends라는 문법은 애초에 extends A,B 이런식으로 다중상속 자체가 성립이 안 되게 만들었습니다. 하지만 인터페이스에서는 implements A,B이런 식으로 가능해졌죠.. 근데 여기서 불편한 점이 하나 더 있었습니다. 그것은 인터페이스는 추상메소드만 작성할 수 있기 때문에 다중상속처럼 여러가지 인터페이스를 지정해도 결국 자식클래스에서는 다 일일이 구현을 해야해서 그 의미가 퇴색되었죠.. 하지만 JDK8버전부터는 인터페이스에 메소드 앞에 default를 붙이면 일반메소드처럼 바디를 구현하여 작성할 수 있게 되었습니다. 사실상 JAVA 에서 다중상속을 허용한 것이죠. 그럼 이제 다중상속을 하였으니 모호성이라는 문제도 같이 생기겠죠?

**모호성**

하나의 자식이 여러 부모를 상속받을 때 부모 필드에 동일한 이름의 필드가 있다면 어떤 부모의 필드인지 알 수 없다 .이를 모호성이라고 합니다.

이를 위해 두개의 인터페이스를 만들어보겠습니다.

```
package test;

public interface InterA {
	default void printData() {
		System.out.println("InterA");
	}
}
```

```
package test;

public interface InterB {
	default void printData() {
		System.out.println("InterB");
	}
}
```

InterA 와 InterB를 만들었는데요, 둘다 똑같은 이름의 메소드 printData()가 있죠? 이제 문제로 가볼게요

**모호성 해결 방법**

\- 상황1 : 두 개의 인터페이스 내에 같은 이름, 같은 매개변수의 메소드가 선언되어 있다.

\- 해결1 : 지정받은 클래스에서 재정의하면 됩니다.

```
package test;

public class ClassA implements InterA, InterB{
@Override
public void printData() {

	InterA.super.printData();
}

public static void main(String[] args) {
	new ClassA().printData();
}
}
```

이런 식으로 두개를 지정하면 InterA.Super. 으로 둘중 어떤 클래스에 있는 메소드를 사용할지 정할 수 있습니다.

![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/2.png)

잘 해결 됐습니다.

\- 상황2 : 부모 클래스의 메소드와 인터페이스의 디폴트 메소드의 이름, 매개변수가 같다.

\- 해결2 : 부모 클래스에 있는 메소드가 사용된다.

```
package test;

public class ClassB {
	public void printData() {
		System.out.println("부모클래스");
	}
}
```

```
package test;

public class ClassC extends ClassB implements InterA{//우선순위는 클래스 상속
	public static void main(String[] args) {
		new ClassC().printData();
	}
}
```

이런식으로 클래스 B와 인터페페이스 InterA에 있는 메소드가 이름이 같은데 클래스를 상속하고 인터페이스를 지정을 같이 한다면 어떻게 될까요?

이럴 때는 우선순위가 정해져있습니다. 바로 클래스죠. 따라서

![Desktop View](/assets/img/Programming-Language/Java/AbstractClass-Interface/3.png)

클래스 ClassB에 있는 printData메소드가 실행되는 것을 볼 수 있습니니다.

이번시간에는 추상클래스와 인터페이스에 대해서 썼습니다!