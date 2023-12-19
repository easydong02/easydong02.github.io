---
title: "[Java] 상속(Extends) 형변환(Casting)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 상속과 객체 간 형변환을 알아보겠습니다. 상속은 문법은 아는데 어떤 기능을 java에서 제공하는 지에 대해서 보겠습니다.

먼저 저는 인류를 3가지 인종으로 나누어서 각각 흑인,백인,황인 클래스를 만들겠습니다. 그런데 각각 멤버변수와 메서드들을 일일이 다 만들어야 할까요? 그렇다면 우리는 프로그래밍의 본질을 잃어버리는 것입니다. 중복을 최대한 줄여서 효율적으로 유지보수가 가능한 클래스들을 만드려면 상속을 해야겠죠? 그렇다면 저 3가지 클래스들을 전부 품을 수 있는 부모클래스인 Man을 만들고 나머지 클래스들을 상속시키겠습니다.

먼저 각 인종의 공통점은 역시 직립보행과 언어를 구사한다는 점이죠. 따라서 이 공통적인 것은 Man클래스에 넣겠습니다.

```
package com.koreait.human;

public class Man{
	//백인,흑인,동양인의 모든 특징을 보유한 보편적 특징.
	String color;
	int age;

	public Man(){;} //기본 생성자만들기 or 아니면 super()에 원래있는 생성자 매개변수 맞춰주기

	public Man(int age){
		System.out.println("Man생성자 호출");
        this.age = age;
	}

	public void walk(){
		System.out.println("걸을 수 있다.");
	}
	public void talk(){
		System.out.println("말할 수 있다.");
	}
}
```

기본생성자와, 나이를 매개변수로 하는 직접생성자 2개를 만들었습니다. 그리고 walk(), talk()라는 메서드 2개를 넣었죠. 이제 각 인종별로 클래스를 만들겠습니다.

//흑인

```
package com.koreait.human;

public class BlackMan extends Man{
	/*클래스안에 올수있는 프로그래밍 언어의 2가지 요소
	 변수(상태),메서드(기능) 
	 */
	String color= "Black";
	int strength =100;

	public void run(){
		System.out.println("흑인은 빨리 뛸 수 있다.");
	}

}
```

흑인만의 특징은 이 클래스에서 담도록 하겠습니다. 일단 흑인은 힘이 좋고 육상에 장점이 있기 때문에 run()메소드와 int 형의 strength를 만들었습니다. 그리고 제일 중요한 상속을 받아야겠죠? 따라서 클래스명 뒤에 extends Man을 넣었습니다.이제 이 클래스는 Man과 상하관계로 놓입니다.

//황인

```
package com.koreait.human;

public class YellowMan extends Man{
	/*클래스안에 올수있는 프로그래밍 언어의 2가지 요소
	 변수(상태),메서드(기능) 
	 */
	String color= "Yellow";
	int height =173;
}
```

황인은 키가작다..(인종차별 아닙니다.ㅜ 그냥 객관적 지표로..)라는 특성을 살려 int 형 height를 만들었습니다.

//백인

```
package com.koreait.human;

public class WhiteMan extends Man{
	
	public WhiteMan(){
		super(3);
	
	String color= "white";
	String noseSize="XL";


}
```

백인을 만들었습니다. 백인은 코가 크니까 noseSize="XL";을 만들었습니다~ 그리고 여긴 직접생성자를 만들었네요 super(), 즉 부모 생성자에 매개변수값 3을 넣었네요? 이것은 부모클래스의 직접 생성자를 호출하는 것입니다.

//UseMan

```
package com.koreait.human;
class UseMan{
	public static void main(String[] args){
		//백인을 생성해보자!
		WhiteMan wm=new WhiteMan();
		System.out.println(wm.color);
		wm.walk();
		wm.talk();
        System.out.println(wm.age);
	
	}
```

이제 이 클래스들을 사용할 UseMan클래스를 만들었습니다. 먼저 백인클래스의 객체 wm을 만들었습니다. 그런데 자세히 보면 잉? walk()메서드와 talk()메서드를 wm이라는 객체를 사용하여 호출하고 있습니다. 이 두 메서드는 백인 클래스가 아니라 Man클래스에 있었는데 말이죠. 답은 상속과 생성자에 있습니다. 백인이 부모 클래스인 Man클래스의 상속을 받는 순간 백인 생성자엔 super();가 쓰여지지 않아도 컴파일러에 의해서 자동으로 호출됩니다. 이 super(); 의 정체는 바로 그 클래스의 부모클래스의 생성자입니다. 따라서 Man(); 이라는 생성자가 먼저 호출 된 다음에 자식클래스 생성자가 호출이 되어서 자식 클래스에서 부모클래스에 접근이 가능하고 사용도 가능하다는 것이죠. 여기선 Man클래스에 직접생성자와 기본생성자 모두를 만들었습니다. 직접생성자를 쓰고싶으면 매개변수에 맞게 값을 입력하면 되죠. 그래서 백인 생성자에 super(3)이런식으로 int age에 맞춰서 사용합니다. 신기하죠?그럼 어떻게 되는지 볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Extends-Casting/1.png)

제가 부모클래스가 먼저 생성되어야하니까 생성자도 먼저 생성되었습니다. 그래서 Man클래스의 직접 생성자안에 있는 출력문이 먼저 나왔구요. 그다음 wm.color와 walk(), talk()가 작동을하고 Man클래스에 있는 멤버변수 age 에 3이 나왔네요. 이것은 백인 클래스의 생성자에 부모클래스 생성자를 건드렸기 때문입니다. 좀 복잡하죠? 하나씩 차근차근 보다보면 쉽습니다!

```
public class Bird{

String name = "새";
public void fly(){
     System.out.println("새가 날아요.");
}

}

public class Duck extends Bird{
 String name= "오리";
 public void fly(){
     System.out.println("오리가 날아요.");
}
}

class Use{
  public static void main(String[] args){
Bird b = new Duck();
System.out.println(b.name);
System.out.println(b.fly());
}
}
```

제가 2개의 클래스를 만들었습니다 새와 오리죠. 오리는 새의 종류니까 부모클래스는 새 자식클래스는 오리로 정했습니다. 각각 같은 이름의 멤버변수 name과 fly()메서드가 있습니다. 근데 여기서 객체 선언이 좀 이상하죠? Bird b = new Duck()를 해버렸네요. 그럼 Bird타입의 객체 b 에 Duck타입의 생성자를 호출해서 넣었습니다. 부모 타입에 자식타입을 넣는것을 업캐스팅이라고 합니다. 다운 캐스팅은 이 업캐스팅 된 객체 b를 다시 강제형변환 (Duck)으로 되돌리는 것이죠. 그럼 여기서 문제는 저 출력문 2개가 있는데 b.name과 b.fly()는 어떻게 출력이 될까요? Java를 만든 Sun사에서는 멤버변수는 부모클래스 기능을 담당하는 메서드는 자식클래스가 호출되도록 설계했죠. 따라서 객체 b는 일단 Bird타입이니까 b.name은 "새"가 출력 될 것이고, b.fly()는 "오리가 날아요."출력문이 나오게 되는 것이죠. 부모타입의 객체를 사용했는데 자식 클래스의 기능이 나오는 것을 보고 오버라이딩이라고 합니다. 다형성의 한 파트이죠.

이번 시간에는 여기까지 하겠습니다!