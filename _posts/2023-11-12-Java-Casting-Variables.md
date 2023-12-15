---
title: [Java] 캐스팅 변수
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

안녕하세요 ^^ 이번 시간에는 up casting과 down casting, 그리고 변수 부분을 다뤄보도록 하겠습니다.

먼저 들어가기 전에 대전제를 알려드릴게요.

"모든 자식은 부모 타입이다." 저번 시간에 상속에 대해 배우면서 부모와 자식 클래스에 대해서 배웠는데요. 모든 자식클래스는 부모타입입니다. 그래서 up casting이 되는 것이지요.

**Casting**

up casting : 자식 값을 부모 타입으로 형 변환하는 작업입니다.

down casting : up casting 객체를 자식 타입으로 형 변환하는 작업입니다. 여기서 단순히 up casting에 반대되는 개념인가? 라고 생각할 수 있지만 부모 값을 자식타입으로 형 변환하는 것은 있을 수 없습니다. 한번더 강조하겠습니다!

\*\* 부모 값은 자식 타입에 담을 수 없다.

그럼 왜 굳이 자식 값을 부모 타입으로 바꾸는 걸까요? 이제 casting을 하는 이유에 대해서 알아보겠습니다.

모든 자식 값을 전달받기 위해서는 동일한 타입의 저장공간으로 받아야 합니다. 하지만 자식끼리는 서로 타입이 다르기 때문에 한 번에 전달받을 수가 없는데요. 이 때 UP CASTING을 하게 되면, 모든 자식이 부모 타입이므로 하나의 저장공간으로 여러 자식을 받을 수 있게 됩니다.

만약 up casting으로 값을 전달 받았다면 자식에서 구현한 기능들을 사용할 수 없기 때문에 다시 down casting을 통해서 복구하여 사용하면 됩니다.

즉, 여러 자식 클래스를 효율적으로 관리하기 위함이라고 생각하시면 됩니다.

이제 한번 형변환을 해볼까요?

Car(부모클래스)를 만들어보겠습니다.

```
class Car{
	
	String brand;
	String color;
	int price;
	
	public Car() {;}
	
	public Car(String brand, String color, int price) {
		super();
		this.brand = brand;
		this.color = color;
		this.price = price;
	}
	void engineStart() {
		System.out.println("열쇠로 시동 킴");
	}
	
	void engineStop() {
		System.out.println("열쇠로 시동 끔");
	}


}
```

전에 했던 것과 같이 했습니다. 이제 자식클래스인 SuperCar 클래스를 만들어보겠습니다.

```
class SuperCar extends Car{
	String mode;
	public SuperCar() {;}
	public SuperCar(String brand, String color, int price, String mode) {
		super(brand, color, price);
		this.mode = mode;
	}

	@Override
	void engineStart() {
		super.engineStart();
		System.out.println("음성으로 시동 킴");
	}
	
	void openRoof() {
		System.out.println("뚜껑 열림");
	}
	void closeRoof() {
		System.out.println("뚜껑 닫힘");
	}
}
```

부모클래스를 상속받고 오버라이딩으로 engineStart 메소드에 "음성으로 시동 킴"을 넣었습니다.

이제 메인 클래스와 메인 메소드를 작성하면서 casting을 해보겠습니다.

```
public class CastingTest {

public static void main(String[] args) {
	
	Car matiz = new Car();
	SuperCar ferrari = new SuperCar();
	//up casting
	Car noOptionFerrari = new SuperCar();
	
	noOptionFerrari.engineStart();
	
	//오류
	
//	SuperCar brokenFerrari = (SuperCar)new Car();
	
	//down casting: 복원, 복구
	SuperCar fullOptionFerrari= (SuperCar)noOptionFerrari;
	fullOptionFerrari.openRoof();
	

}

}
```

먼저 Matiz라는 car타입의 객체와 ferrari라는 supercar타입의 객체를 만들었습니다. 여기까진 하던대로 가는데..

이제 up casting을 해볼게요. 먼저 car타입의 객체 noOptionFerrari를 만들었는데 잉? 여기에 대입된 생성자는 SuperCar 입니다. 생성자는 메소드가 아닌 값이라고 했죠? 즉 필드를 메모리에 할당하고 그 주소값을 갖고있는 하나의 '값'입니다. 그리고 대입연산자로 부모타입의 객체로 넣었습니다.

그리고 noOptionFerrari. 을 쳐보면 openroof는 나오지 않습니다. 부모타입이라 supercar 클래스에만 있는 것들은 구현되지 않기 때문이죠. 물론 오버라이딩으로 필드의 주소값을 재정의하면 사용할 수는 있습니다.

그 다음은 자식타입의 fullOptionFerrari 객체를 만들고 여기에 noOptionFerrari 를 넣었습니다. 하지만 그 전에 형변환으로 (SuperCar)타입으로 바꾸었네요. 다시 down casting으로 복원하였으니 supercar클래스에만 있는 openroof 메소드를 사용할 수있는 것도 보일겁니다.

**객체 간 타입 비교**

instanceof

a instanceof B : 참 또는 거짓, 둘 중 하나의 값으로 변환됩니다.

\- a가 B타입이면 true

\- a가 B타입이 아니면 false

이거로 한번 위의 클래스들을 비교해볼까요?

```
    matiz instanceof Car	: true
	matiz instanceof SuperCar : false
	ferrari instanceof Car : true
	ferrari instanceof SuperCar : true
	noOptionFerrari instanceof Car : true
	noOptionFerrari instanceof SuperCar : true
	fullOptionFerrari instanceof Car : true
	fullOptionFerrari instanceof SuperCar : true
```

마티즈는 부모타입이므로 supercar 타입에는 false가 나옵니다. 반면에 ferrari는 자식클래스이므로 자식타입 뿐만 아니라 부모타입에도 true가 나옵니다. 왜냐면 위에 강조했죠? 모든 자식은 부모타입이다.

그 다음은 noOptionFerrari네요. up casting되었는데요, 부모타입 true는 당연한데 신기한 것은 up casting이 되어도 결국은 자식 클래스라 supercar타입에도 부합합니다. down casting된 fullOptionFerrari도 마찬가입니다.

**변수의 종류**

지역변수(local variable) : 지역 안에서 선언된 변수(클래스 지역은 제외)

매개변수(parameter) : 메소드 선언 시 소관호 안에서 선언되는 변수

전역변수(global variable) : 전 지역에서 사용할 수 있는 변수(클래스의 인스턴스 변수)

정적변수(static variable) : 객체 간 공유, 편의성

지역변수 매개변수 전역변수는 많이 들어봤죠? 자세한 설명은 안 하겠습니다.

여기서 자세히 볼 것은 정적변수인데요, 객체 간 공유? 이게 무엇일까요

이를 알기 위해서 같은 패키지 안에 아래와 같이 2개의 클래스를 만들었습니다.

![Desktop View](/assets/img/Programming-Language/Java/Casting-Variables/1.png)

그리고 각각 클래스에 소스코드를 보여드릴게요

Variable1

```
package variable;

public class Variable1 {
int data;
static int data_s;
}
```

Variable2

```
package variable;

public class Variable2 {

	public static void main(String[] args) {
Variable1 v1 = new Variable1();
		v1.data++;
		v1.data++;
		v1.data++;
		v1.data++;
		v1.data++;
		v1 = new Variable1();
		v1.data++;
		v1.data++;
		v1.data++;
		v1.data++;
		v1.data++;
		System.out.println(v1.data);
```

Variable2클래스에서 Variable1 에 있는 클래스 생성자로 객체 v1을 만들었습니다. 이제 data를 1씩 증가시키고 다시 중간에 같은 이름 v1으로 생성자를 재정의하고 다시 1씩 증가시켰습니다. 마지막 출력 v1.data에는 어떤 값이 나올까요? 10? 5? 답은 5입니다. 생성자를 재정의한 순간 다른 주소값이 v1으로 들어가게 되고 따라서 그 주소값에서 새로 쓰여지는 거죠. 그래서 0부터 다시 시작해서 5가 나오는 겁니다.

이제 정적변수로 가볼까요? Variable1에 static, 즉 정적변수로 선언된 data\_s를 사용해봅시다.

```
package variable;

public class Variable2 {

	public static void main(String[] args) {
Variable1 v1 = new Variable1();
Variable1 v2 = new Variable1();
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		v1 = new Variable1();
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		v1.data_s++;
		System.out.println(v1.data);
        System.out.println(v2.data);
```

아까와 마찬가지로 설정했습니다. 마지막 출력은 5가 나올까요? 답은 10입니다. 정적변수로 선언한 순간 생성자가 아닌 컴파일러가 메모리에 올려버리기 때문에 생성자를 몇개를 중복 재정의해도 값은 변하지 않습니다. 심지어 v2라는 다른 객체로 접근해도 똑같이 10이 나옵니다. 따라서 객체나 생성자는 정적변수에 애초에 의미가 없기 때문에 정적변수를 표현할 때는 그 클래스를 적어버립니다.

```
Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		v1 = new Variable1();
		Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		Variable1.data_s++;
		System.out.println(Variable1.data_s);
```

이런식으로 말이죠.

**저장 기억 부류(storage class)**

Stack영역 Data영역

\-----------------------------------------------------------------------------

지역변수, 매개변수 전역변수 정적변수

**초기화** 직접 자동 자동

**생명주기** } new 프로그램 종료 시

지역변수와 매개변수는 메모리에서 Stack영역으로 들어갑니다. 그리고 전역변수와 정적변수는 Data영역으로 들어갑니다.

초기화는 지역변수와 매개변수는 직접 해야합니다. 반면에 전역변수와 정적변수는 자동으로 해줘요! 생명주기는 Life Cycle 입니다. 지역변수와 매개변수는 그 지역끝나면 메모리에서 해제됩니다. 어떤 메소드에서 선언된 변수는 그 메소드가 끝나는 }를 만나면 해제되겠지요? 전역변수는 위에서 data예시를 들때 사용했습니다. new 연산자로 새로운 주소값이 객체에 재정의 되었을 때 초기화 되고 값이 5로 나오는 것을 확인할 수 있었습니다. 정적변수는 항상메모리에 남아있기 때문에 프로그램 종료가 되어야 메모리에서 해제가 됩니다.

**접근 권한 제어자(접근자)**

default : 다른 패키지에서 접근할 수 없습니다. 그냥 변수를 선언하면 이 default로 설정됩니다.

public : 모든 곳에서 접근할 수 있습니다. .java파일의 대표 클래스 표시

protected : 다른 패키지에서 접근할 수 없습니다. 단 자식은 접근 가능

private : 다른 클래스에서 접근할 수 없습니다. 직접 접근이 아닌 간접 접근을 위해서 사용합니다.

이번 시간은 여기까지 하도록 하겠습니다~