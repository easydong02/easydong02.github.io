---
title: "[Java] 메서드 은닉화 다형성 자바Doc"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 메서드와 은닉화, 오버로딩, 생성자를 다시 리뷰해보겠습니다. 굵직한 주제들이라 어떻게 잘 전달해야 할지 벌써부터 고민이군요..

설명은 이미 충분히 했으니 이번에는 각 주제에 맞는 코드를 보면서 중요한 것들을 짚어볼게요

메서드에선 3개의 클래스를 만들어 각 클래스를 유기적으로 연결을 메서드를 통해서 해보겠습니다.

**Hero클래스**

```
package com.koreait.hero;

import com.koreait.bullet.Bullet;

public class Hero {
	private int hp=10;
	private boolean fly=false;
	private String name="메가맨";
	
	public void setHp(int hp) { //hp 값을 변경하고 싶다  
		this.hp=hp;
	}
	public int getHp() { //hp 값을 변경하고 싶다  
		return hp;
	}
	public void setFly(boolean fly) {//fly 값을 변경하고 싶다
               this.fly = fly;
	}
	public boolean getFly() {//fly 값을 변경하고 싶다
               return fly;
	}
	public void setName(String name) {//name 값을 변경하고 싶다		
             this.name= name;
	}
	public String getName() {//name 값을 변경하고 싶다		
             return name;
	}
	public void fire(Bullet b){//bullet 을 다른 무기로 변경하고 싶다
             System.out.println("내가 사용하는 무기는 : " + b.getWeapon());
	}
}
```

먼저 멤버변수 int타입의 hp와 boolean타입의 fly, 마지막으로 String 타입의 name을 만들고 각각 초기값을 주었습니다. 그리고 각 멤버변수를 private 접근제어자를 두어서 각각 getter, setter메서드를 만들어주어서 변경과 반환을 할 수 있도록 하였습니다. 그리고 fire()이라는 메서드를 만들었습니다. 밑에서 작성할 Bullet클래스를 가져오는 메서드입니다. 물론 Bullet클래스는 다른 패키지에 있으므로 임포트까지 적어주는 모습입니다.

**Bullet클래스**

```
package com.koreait.bullet;


public class Bullet{
	private String weapon="칼";

	public void setWeapon(String weapon){
		this.weapon = weapon;
	}
	public String getWeapon(){
		return weapon;
	}
}
```

Bullet클래스를 만들었습니다. String 타입의 weapon을 만들어 초기값 "칼"을 넣었습니다. 이 역시 getter, setter를 만들었습니다.

이제 이 클래스들을 실질적으로 사용할 클래스를 만들어봅시다.

**UseHero클래스**

```
package com.koreait.use;


import com.koreait.hero.Hero;
import com.koreait.bullet.Bullet;

class UseHero{
	public static void main(String[] args){
		Bullet b = new Bullet();
		Hero hero = new Hero();
		hero.setName("배트맨");
		hero.setHp(100);
		hero.setFly(true);
		System.out.println("히어로 이름: " +hero.getName()); 
		System.out.println("히어로 체력: " +hero.getHp());
		System.out.println("나 날고있나? : " +hero.getFly());
		
		b.setWeapon("텀블러");
		hero.fire(b);	
	}
}
```

앞의 2개의 클래스를 모두 사용할거라 모두 임포트를 해주었습니다. Bullet타입과 Hero타입의 객체를 각각 만들었습니다. 그리고 히어로 타입에 제가 좋아하는 히어로 정보를 넣었습니다. 여기서 중요한건 hero.fire(b);구간입니다. Hero클래스에 있는 fire()메소드를 사용합니다. 그리고 그 메서드 안에는 Bullet클래스의 weapon을 반환하는 메서드입니다. 그리고 배트맨은 칼보다 텀블러를 사용하기 때문에 객체 b를 이용해서 weapon값을 변경하였습니다. 이처럼 데이터에 private접근제어자를 두고 메서드를 이용해서 값을 변경 호출하는 방법을 은닉화(encapsulation)라고 합니다. **그리고** **hero.fire(b); 에서 괄호 안에 new Bullet()을 넣어도 될까요? 당연히 됩니다. 왜냐하면 결국 그 소괄호 안에는 Bullet타입의 주소값이 있으면 되거든요. 다만 new Bullet()을 해버리면 이미 존재하는 인스턴스 b와는 다른 새로운 주소값이겠죠? 따라서 초기값 "칼"이 나오겠습니다.**

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/1.png)

잘 출력되는 것을 볼 수 있었습니다. 이클립스로 하는 것보다 살짝 원시적으로 사용하는게 제가 뭔가를 더 확실히 배울 수 있어서 좋았습니다.

이번엔 다른 모범(?)클래스를 가지고 오버로딩에 대해서 알아보겠습니다!

**Bird클래스**

```
package com.koreait.animal;

public class Bird{

	//메서드들 간의 로직의 큰 차이가 없을에도,
	//새로운 메서드명을 정의해야 한다면, 오히려
	//개발에 장애요인이 될 수 있다.. 따라서 sun에서는 하나의 클래스 내에 메소드명을
	//중복정의할 수 있는 기법을 제공해준다.. 이를 가리켜
	//메서드 오버로딩(OverLoading), 단 오버로딩을 인정해주는 조건
	//메서드명은 같되, 매개변수의 자료형 및 개수는 달라야한다...
	public void fly(){
		System.out.println("새가 날아간다.");
	}
	
	//조금 더 높이 날자..
	public void Fly(int height){
		System.out.println("새가 조금 더 높이 날아간다.");
	}

	public void Fly(String stop){
		System.out.println("새가 더욱 조금 더 높이 날아간다.");	
	}
}
```

클래스를 하나 만들었습니다. 여기서 메서드 fly()를 만들었는데 이와 비슷한 다른 메서드를 만들 때 또 심사숙고해서 만든 메서드이름을 바꾸어야할까요? 아닙니다 우리에겐 오버로딩기능이 있죠. 이것은 메서드의 매개변수가 다르다면 같은 이름이라도 중복허용을 한다는 것이죠. 따라서 그 메서드에 매개변수에 어떤 것을 넣느냐에 따라 어떤 메서드가 호출이 될지 결정을 하죠. 참 편리한 기능입니다!

따라서 이 클래스를 사용하는 클래스에선 원하는 메서드를 호출하기 위해 그에 따른 매개변수를 입력하면 되겠습니다!

이제는 또 다른 클래스를 보면서 생성자에 대해서 알아보겠습니다.

**Car클래스**

```
package com.koreait.traffic;

public class Car{
	String color;
	int speed;
	int price;
	
    public Car(){;}

	public Car(String color, int speed, int price){
	this.color =color;
	this.speed = speed;
	this.price = price;
	}

	public void main(String[] args){
		Car c =new Car("red",400,20000);
	    Car c2 = new Car();
	}

}
```

보시면 클래스에 클래스이름과 같은 메서드가 2개 나오죠? 이것을 설명하기 전에 메인메서드에서 Car c2 = new Car();을 보겠습니다. 우리가 보통 객체를 사용할 때 쓰는 문장인데, Car()부분은 메서드를 선언하는 것일까요 아님 사용하는 것인가요? 답은 사용입니다. 그런데 모든 코드가 그렇듯이 사용을 하려면 그 전에 선언을 해야 사용을 하는데 어디서 선언이 되었을까요? 그리고 이 메소드의 정체는 무엇일까요? 클래스이름과 같은 메서드인데 타입(void, int, String)이 없는 메서드를 생성자라고 합니다. 그리고 클래스 내에서 생성자를 선언하지 않으면 java에서는 디폴트 생성자를 컴파일 할 때 만들어 줍니다. 그리고 우리가 직접 생성자를 만들면 이 디폴트 생성자는 만들어지지 않습니다. Car타입의 객체 c를 봅시다. c에는 new Car("red",400,20000); 가 들어있습니다. 즉

```
	public Car(String color, int speed, int price){
	this.color =color;
	this.speed = speed;
	this.price = price;
	}
```

우리가 직접 만든 이 생성자에 순서대로 매개변수를 넣어서 우리가 객체를 만들고 만들어진 객체에서 객체와 메서드를 사용하여 그 멤버변수를 일일이 수정할 필요없이 출고되면서 동시에 저 속성들이 부여되는 것이라고 보면 됩니다. 편하죠? 그런데 이 직접생성자를 만들면 기본생성자는 쓸 수가 없기 때문에 만들어 줍니다. 그리고 그 우리가 만든 기본생성자는 public Car(){;}입니다!

따라서 생성자를 선언하여 객체를 만들면서부터 개성을 부여할 수 있습니다! 재밌네요 ㅎㅎ

**JAVADOC**

실무로 들어가보겠습니다. 실무에서 만약 어떤 프로그램을 주고 받을 때 원본 코드가 들어있는 java파일을 주고 받을까요? 절대 아닙니다. 그것은 그 회사의 기술을 고대로 넘겨주는 일입니다. 따라서 컴파일하여 바이너리 코드로 이루어진 .class파일을 주는 것이죠. 그러면 우리가 이 클래스파일을 받는 입장이라고 합시다. class파일을 어떻게 열어서 안에 무슨 생성자가 있는지, 어떤 메서드가 있는지, 어떤 멤버변수가 있는지 알 수 있을까요? 모릅니다. ㅋㅋ 우리가 기계가 되어서 기계어를 읽지 않는 이상.. 따라서 받은사람을 위해서 read me 같은 설명서를 주어야 합니다. 그런데 어떻게 적을까요? 사실 이걸 해주는 프로그램이 있습니다. 바로 java bin에 있는 javadoc입니다. 한번 써봅시다.

```
package com.koreait.member;

import com.koreait.animal.Dog;

public class Smith{
	String name= "스미스";
	public int sal = 800;
	/**
	이 메서드는 불쌍한 우리 스미스의 급여를 변경하기 위함임

	**/
	public void setSal(int sal){
		this.sal = sal;
	}
	public int getSal(){
		return sal;
	}
	//Dog 도 자료형이므로, 당연히 매개변수로 가능.
	public void love(Dog d){
		System.out.println("내 강아지 이름은 : " + d.getName());
	}
}
```

예전에 썼던 스미스 클래스입니다. 이제 cmd 창으로 가봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/2.png)

한줄 씩 보자면 먼저 스미스가 있는 member폴더를 cd명령어로 기본 경로로 지정했습니다. 그다음엔 javadoc프로그램을 사용할거라 javadoc를 적어줍니다. 그리고 한칸 띄고 Smith.java를 치고 엔터를 누르면 뭔가가 쭈루룩 나옵니다. 이제 member폴더로 가봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/3.png)

뭐가 여러가지가 엄청 생겼습니다 ㅎㅎ 이제 저기서 가장 메인 웹페이지인 index를 들어가봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/4.png)

우리가 만들었던 Smith클래스에 대한 정보가 java 공식 홈페이지의 ui대로 나와있네요.. 정말 신기합니다. 들어가볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Method-Capsule-Overloading-Generator-JavaDoc/5.png)

저렇게 어떤 멤버변수가 있는지 메서드가 있는지 나옵니다. 물론 어노테이션 주석( /\*\*~ \*/)으로 우리가 원하는 주석을 직접 넣어줄 수 도 있습니다. 참 신기하죠?

이번 시간에는 여기까지 하겠습니다!