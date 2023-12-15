---
title: [Java] 익명클래스 예외처리
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


이번 시간에는 익명클래스와 예외처리를 알아보겠습니다.

**익명 클래스(Anonymous inner class)**

이름이 없는 클래스이며 구현되지 않은 필드를 구현하기 위해 일회성으로 생성되는 클래스입니다.

바로 예시를 만들어보겠습니다. 다들 스타벅스 아시죠? 그 스타벅스의 인터페이스를 만들어 메뉴를 담을 문자열 배열타입의 메소드와 판매방식의 틀을 만들어 보겠습니다.

```
package anonymous;

public interface Form {
	public String[] getMenu();
	public void sell(String order);
}
```

인터페이스 Form을 만들고 구현되지 않은 추상필드 2개를 만들었습니다.

이제 스타벅스의 클래스를 만들어 여기서 각 지점이 각 클래스에서 사용할 메소드를 만들겠습니다.

```
package anonymous;

import java.util.Scanner;

public class Starbucks {
	Scanner sc = new Scanner(System.in);
	public void register(Form form) {
		String[] arMenu = form.getMenu();
		System.out.println("======메뉴판======");
		for (int i = 0; i < arMenu.length; i++) {
			System.out.println(i + 1 + ". " + arMenu[i]);
		}
		if(form instanceof FormAdapter) {
			System.out.println("무료 나눔중...");
		}else {
			form.sell(sc.next());
		}
		
	}
}
```

스타벅스 클래스를 만들었습니다. 그리고 register라는 void타입의 메소드를 만들었습니다. 여기서 매개변수는 Form타입의 객체입니다. Form타입의 객체가 들어오면 arMenu라는 문자열 배열에 Form에 있는 getMenu메소드를 대입합니다. 그리고 메뉴판이라는 문자열을 출력하고 for문을 만듭니다. 그리고 여기선 arMenu에 들어간 getMenu 리턴값들을 하나씩 출력합니다. 그리고 if문이 보이는데요 이것은 나중에 설명 드리겠습니다. 그리고 Form에서 sell 메소드에 매개변수에 메뉴 이름을 넣을 스캐너를 썼습니다. 여기까지 보면 뭐가 뭔지 하나도 감이 안 잡히시죠? 몇개 더 있으니까 차근차근 보여드리겠습니다!

```
package anonymous;

public abstract class FormAdapter implements Form {

	@Override
	public String[] getMenu() {
		
		return null;
	}

	@Override
	public void sell(String order) {;}

}
```

저번시간에 배운 어댑터를 만들었습니다. Form에 있는 메소드들을 강제로 모두 쓸 필요가 없는 경우가 생기기 때문에 추상클래스 FormAdapter를 만들고 인터페이스 Form을 지정했씁니다. 그리고 재정의를 통해 구현했죠

이제 각 스타벅스의 각 지점을 만들어보겠습니다.

```
package anonymous;

public class Road  {
	public static void main(String[] args) {
		//무료 나눔 행사 중...
		//sell메소드를 구현할 필요 없음.
		//무료 나눔 행사중인 매장은 주문 완료 대신 "무료 나눔중!" 출력
		Starbucks jamsil = new Starbucks();
		jamsil.register(new FormAdapter() {
			
			
			@Override
			public String[] getMenu() {
				String[] arMenu = {"아메리카노", "딸기주스", "에스프레소"};
				return arMenu;
			}
		});
	
		Starbucks gangnam = new Starbucks();
		gangnam.register(new Form() {
			
			@Override
			public void sell(String order) {
				String[] arMenu = getMenu();
				for (int i = 0; i <arMenu.length; i++) {
					if(arMenu[i].equals(order)) {
						System.out.println(order + " 주문완료");
                        break;
					}
					
					
				}
			}
			
			@Override
			public String[] getMenu() {
				String[] arMenu = {"아메리카노", "카페라떼", "에스프레소"};
				return arMenu;
			}
		});
	}
}
```

좀 길죠? 천천히 봐볼게요. 일단 여기서 메인메소드를 만들었습니다. 제가 스타벅스 지점을 2개 만들었습니다. 잠실점, 그리고 강남점인데요 잠실점은 Form인터페이스에 있는 sell메소드를 쓰지 않겠습니다. 일단 jamsil이라는 객체를 만들어 스타벅스 클래스의 필드 주소값을 넣은 다음에 스타벅스의 register 메소드를 사용하겠습니다. 근데 보이시나요? 사용을 하긴 했는데 매개변수자리에 좀 긴 코드가 보이죠? 보통 매개변수엔 어떤 값을 넣기 때문에 긴 코드가 나올필요가 없는데... 저게 바로 익명클래스입니다. 사용을 해야하는 매개변수에서 구현을 하고있는 모습이 보입니다. 왜냐하면 구현되지 ㅎ않았기 때문에 구현을 해야하는데 메소드를 사용할때만 일회용으로 만든 것입니다. 그래서 jamsil이란 스타벅스 클래스의 객체에서 register라는 클래스를 사용함과 동시에 일시적으로 구현을 한거죠. 여기엔 sell메소드는 구현하지 않고 getMenu메소드만 구현했습니다. 만약 매개변수에 Form타입의 주소값을 넣었다면 불가능했지만 어댑터를 통해서 선택적으로 구현하게 만들었습니다.

.. 어렵죠? 저도 너무 어려웠어요 하지만 보고 또 보고하니까 조금씩 머리에 들어오는게 느껴지고 그게 재밌더라고요.. 처음부터 다 이해하고 써먹을줄 알면 재미없잖아요 ㅎㅎ

그 다음은 강남지점입니다. 여기는 sell메소드도 사용할거기 때문에 코드가 좀 깁니다. 사실 보면 전부 매개변수 ()안에 들어있는 겁니다. 그냥 register라는 메소드를 사용만 하는데 저렇게 코드가 길게 나온거죠.

gangnam이라는 객체를 만들고 마찬가지로 스타벅스 클래스의 필드의 주소값을 객체에 대입했습니다. 그리고 register라는 메소드를 사용하는데 그 매개변수에는 어댑터가 아닌 그냥 Form타입의 필드주소값을 받았습니다. 그리고 sell과 getMenu메소드 둘다 일시적으로 익명클래스로 구현했습니다. sell을 보면 arMenu라는 문자열 배열에 getMenu메소드에 있는 값들을 대입했습니다. 그리고 for문을 통해서 arMenu에 있는 것들이 order와 일치할 때 그것을 주문완료하였다는 출력문을 만들었습니다.

그리고 getMenu는 그 지점의 메뉴를 작성했습니다.

설명이 엄청 길었네요. 결국 크게 보면 Road라는 클래스에서는 각 지점의 객체를 2개 만들고(잠실,강남) 스타벅스클래스에서 만든 register라는 메소드를 각각 사용만 했습니다. 그 사용하는 과정에서 매개변수에 엄청난 것들이 들어간거죠.

이제 실행해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/AnonymousClass-Exception/1.png)

자 첫번째 메뉴판에는 잠실점입니다. 여기는 sell을 구현하지 않았죠? 그냥 이벤트? 같이 무료나눔입니다. 아까 스타벅스 클래스 설명 할때

```
if(form instanceof FormAdapter) {
			System.out.println("무료 나눔중...");
		}else {
			form.sell(sc.next());
		}
```

이런 부분이 밑에 있었죠? 만약 매개변수에 들어온 타입이 그냥 Form이 아니라 어댑터라면 "무료 나눔중..."을 출력하도록했습니다. 그렇지 않으면 sell 메소드에 우리가 주문할 것을 입력하는거죠..

어렵지만 그만큼 재미가 있었습니다!

**예외(Exception) 처리**

예외 처리란 코드를 실행할 때 발생하는 에러가 있을 때 어떻게 수정하고 다른 코드를 실행하여 결국 중간에 에러로 멈추지 않고 코드들을 정상적으로 수행하게 하기 위한 처리방식입니다.

**예외 처리 문법**

try{

오류가 발생할 수 있는 문장

}catch(예외이름 객체명){

오류 발생 시 실행할 문장

}catch(예외이름 객체명){

오류 발생 시 실행할 문장

}

...

}finally{

오류 발생 여부에 상관없이 무조건 실행할 문장

※ 외부 장치와 연결했을 경우 다시 닫을 때 주로 사용된다.

}

try에는 오류가 발생할 수 있는 문장을 적어주고 각 예외(에러)들을 지정하고 이런 예외가 나왔을때 실행할 문장을 catch 후에 오는 중괄호에 적습니다.

**예외 처리를 사용하는 이유**

\- 제어문으로는 처리할 수 없는 경우가 있을 수 있다.

\- 프로그램이 강제 종료(팅김현상)되는 것을 막기 위하여 사용합니다.

이제 한번 만들어 볼까요?

먼저 에러 상황을 만들어 봅시다. 5칸배열에 10번째 인덱스에 값을 넣는 오류를 범해볼거에요

```
package exception;

public class ExceptionTest {
	public static void main(String[] args) {
		int[] arData = new int[5];
		
			try {
				arData[10]= 10;
			} catch (Exception e) {
				System.out.println(e);
			}
   }
}
```

정수타입의 배열 arData를 선언하고 5칸으로 지정했습니다. 그런데 arData\[10\]= 10;를 try문 안에 적어줍시다 역시 오류가 나오겠죠? catch문에서 소괄호에는 예외이름의 타입과 그 객체명을 적었습니다. 자, 여기서 JAVA의 예외를 공부할건데 어떤 코드를 실행했을 때 오류가 있으면 콘솔창에 오류가 뜨죠? 오류가 떴다는 것은 우리가 클래스나 메소드를 배웠던 것처럼 구현이 되어있어야 콘솔창에 출력이 되는 것이겠죠? 마찬가지입니다. JAVA는 에러를 그냥 미지의 오작동이라고 보지 않고 각 에러의 타입을 다 정해서 만들어 놓았습니다~ 신기하죠? 그리고 그 모든 예외타입의 부모타입은 Exception이라는 놈입니다. 악당의 두목같은 개념이라고 생각하시면 됩니다. 그리고 객체명은 우리가 그냥 클래스 객체명을 정할 때처럼 정하면 됩니다. 그렇다면 일단 어떤 오류가 나더라도 Exception이라는 타입에 속하기 때문에 catch문 바디에 있는 코드가 실행이 됩니다. 그게 바로

```
System.out.println(e);
```

입니다. 이것은 그게 어떤 오류인지 정확히 표시해줍니다.

이제 실행해 보죠.

![Desktop View](/assets/img/Programming-Language/Java/AnonymousClass-Exception/2.png)

콘솔창에 오류의 종류를 표시했습니다. 정확한 오류타입은 ArrayIndexOutOfBoundsException였네요 그럼 원래 있던 코드를 수정해보겠습니다.

```
try {
				arData[10]= 10;
			} catch (ArrayIndexOutOfBoundsException e) {
				System.out.println("인덱스의 범위를 벗어나 값을 대입하였습니다.");
			}catch(Exception e) {
				System.out.println(e);
			}
			
		
```

이제 정확한 오류타입을 알았으니 원래 있던 Exception에서 ArrayIndexOutOfBoundsException로 바꿔줍니다. 그리고 실행할 문장에는

"인덱스의 범위를 벗어나 값을 대입하였습니다."라는 문구를 출력했습니다. 따라서 우리가 어떤 오류를 범했는지 알 수 있죠. 근데 코드를 만들면서 에러는 하나만 일어날까요? 절대 아닙니다. 그래서 또 다른 catch문을 만들어서 처음처럼 어떤 오류인지 알아내는 문장을 적었습니다. 이런식으로 하나씩 오류를 알아가고 수정하고 하면서 견고한 코드가 만들어지겠죠? 정말 재밌습니다!

이번시간에는 익명클래스와 예외처리의 개념에 대해서 조금 알아봤습니다!!