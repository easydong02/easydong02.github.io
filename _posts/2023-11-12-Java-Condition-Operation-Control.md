---
title: [Java] 조건식 연산자 제어문
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

이번시간에는 조건식과 몇가지의 연산자와 제어문을 알아보겠습니다.

**\*조건식**

참 또는 거짓 둘 중 하나가 나오는 식입니다.

다른 언어에서의 참, 거짓 값

참 : 1

거짓 : 0

C언어에서는 거짓은 0, 나머지 모든 숫자는 참을 의미 했습니다. 자바는 조금 다릅니다.

JAVA에서의 참, 거짓 값

참: true

거짓: false

자바는 아예 true, false로 정해버립니다. 읽기엔 편하겠네요

true, false를 담을 수 있는 자료형

자료형 type byte 값 초기값

논리형 oolean 1 true,false false

```
boolean isTrue = 10>11;
```

이런 식으로 boolean이란 타입의 itTrue라는 변수를 만들었습니다. 용량은 1byte인데 사실 참,거짓 2가지만 있으면 되기 때문에 컴퓨터가 사용할 수 있는 최소한의 용량을 가져갑니다.

**\*관계 연산자**

\== : 같다

!= : 같지 않다

\>,< : 초과, 미만

\>=,<= : 이상, 이하

많이 익숙하죠? C언어에서와 같습니다 ㅎㅎ 여전히 주의할 점은 대입연산자 =와 관계연산자 ==를 잘 구분해야 하는 것입니다.

**\*논리 연산자**

&&, and : A && B -> A와 B조건식 모두 참일 때 참

||, or : A||B -> A와 B 조건식 둘 중 하나가 참일 때 참

역시 마찬가지입니다.

\***단항 연산자**

!, NOT : true는 false로, false는 true로 변경

```
System.out.println(!(10>11 &&10!=11));
```

!(10>11 &&10!=11) 에서 괄호를 친 이유는 !라는 단항 연산자는 연산자 우선순위에서 상대적으로 나머지 연산자보다 높기 때문에 괄호 처리를 하지 않으면 그냥 !10이 되버리는 겁니다. 따라서 10>11 &&10!=11는 거짓이 나오겠죠? 이 전체를 단항 연산을 하면 참이 나오겠네요 ^^

**\*삼항 연산자**

? :

조건식 ? 참 : 거짓

C언어와 같습니다. 조건식을 ? 전에 기입하고 참과 거짓을 각 조건식의 결과에 따라 실행합니다. 그렇다면 두 정수를 입력받고 큰 값을 출력하는 것은 어떻게 할까요?

```
import java.util.Scanner;

public class OperTest2 {
	public static void main(String[] args) {
		//두 정수 입력받기
		String result ="";
		Scanner sc = new Scanner(System.in);
		System.out.println("첫번째 정수를 입력하세요.");
		int num1 = sc.nextInt();
		System.out.println("두번째 정수를 입력하세요.");
		int num2 = sc.nextInt();
		
		
		result = num1>num2 ? "큰 값 :"+num1 : num1==num2?  "두 수는 같습니다" : "큰 값: "+num2;
		
		System.out.println(result);
	}
}
```

만들어 봤습니다! result라는 문자열을 만들고 초기화를 했습니다. 그래서 정수 2개를 입력하고 그 밑에 삼항 연산자로 두 숫자 중 더 큰 수를 찾아 출력하죠 처음엔 num1>num2 로 시작하고 참이면 큰 값: num1을 출력하는군요. 근데 중요한것은 여기서 거짓이라 해서 num2를 바로 큰수로 이해하기엔 성급합니다. 왜냐하면 두 수가 같을 수도 있기 때문이죠. 따라서 거짓이라면 두수가 같은지 한번 더 물어봅니다. 그리고 또 그 조건식에서 참 거짓을 나누어 코드를 실행합니다.

이제 다른것도 해보죠 혈액형을 숫자로 입력받아 그 혈액형의 특징을 출력하고 싶습니다. 한번 만들어 볼까요?

```
public static void main(String[] args) {
		// 혈액형별 성격
		//당신의 혈액형은?
		//1.A
		//2.B
		//3.O
		//4.AB
		//A : 꼼꼼하고 세심하다. 
		//B : 추진력이 좋다.
		//C : 사교성이 좋다.
		//AB : 착하다.
		
		String qMsg = "당신의 혈액형은? "
				+"\n1. A"
				+"\n2. B"
				+"\n3. O"
				+"\n4. AB";
		String aMsg = "꼼꼼하고 세심하다.",
		 bMsg = "추진력이 좋다.",
		 oMsg = "사교성이 좋다.",
		 abMsg = "착하다.";
		
		String result="";
		
		int 	aBlood =1,
				bBlood =2,
				oBlood =3,
				abBlood =4;
		
		int 	choice =0;
		
		System.out.println(qMsg);
		Scanner sc = new Scanner(System.in);
		
		choice = sc.nextInt();
		
		result = choice ==aBlood?  aMsg : choice ==bBlood? bMsg :
			choice ==oBlood? oMsg : choice ==abBlood? abMsg : "잘못입력하셨습니다.";
		System.out.println(result);
	}
```

아까와 같이 만들었지만 조금 다른점이라면 하단 부분의 코드에는 대부분 미리 선언된 변수를 적는 방식으로 되어있습니다. 이 부분에서 개발자의 능력이 갈린다고 생각합니다. 만약 나중에 저 코드를 변경할 경우 고쳐야할 코드들이 너무나도 많아지기 때문에 일괄적으로 관리하기 쉬운 변수를 만들어 버리는게 좋죠.

이제 여러분은 어떤 것을 느끼고 계시죠? 네 통했네요 ㅎㅎ

삼항연산자를 사용하면너무 코드가 길어지고 사용하는데 한계가 느껴짐을 알 수있습니다.

그럴 땐 If연산자를 사용하면됩니다.

**\-if 문**

if(조건식){

실행할 문장;

}

if(조건식){

}

if(조건식){

}

else if(조건식){

}

else{

}

C언어와 같습니다 . if 는 위의 조건문과는 상관없이 독립적으로 실행됩니다. 반면 else if 와 else는 기본적으로 위의 조건식이 거짓이었을때 코드를 일단 실행하는 문장입니다. else는 모든 조건식이 거짓이어야 실행이 됩니다. 이 것을 이용해서 아까 만든 큰값 구하기를 if문으로 만들어보겠습니다.

```
public static void main(String[] args) {
		// 두 정수 입력받기
		String result = "";
		Scanner sc = new Scanner(System.in);
		System.out.println("첫번째 정수를 입력하세요.");
		int num1 = sc.nextInt();
		System.out.println("두번째 정수를 입력하세요.");
		int num2 = sc.nextInt();

		if(num1>num2) {
			result = "큰 값 : "+num1;
		}
		else if(num1<num2) {
			result = "큰 값 : "+num2;
		}
		else {
			result = "두 수는 같습니다.";
		}
		System.out.println(result);
	}
```

밑에 삼항 연산자부분을 if 로 하니까 편하죠? 코드가 복잡해질수록 제어문이 더 쓰기 좋습니다! result 에 각각의 조건문에서 나오는 값이 다르겠지요? 마찬가지로 두 수가 같을땐 else를 주어서 해결합니다. 이제는 아까 만든 혈액형에서 if 문을 사용해보겠습니다!

```
public static void main(String[] args) {
		String qMsg = "당신의 혈액형은?"
	            + "\n1. A"
	            + "\n2. B"
	            + "\n3. O"
	            + "\n4. AB";
	      
	      int choice = 0;
	      Scanner sc = new Scanner(System.in);
	      
	      String aMsg = "꼼꼼하고 세심하다.";
	      String bMsg = "추진력이 좋다.";
	      String oMsg = "사교성이 좋다.";
	      String abMsg = "착하다.";
	      String errMsg = "다시 시도해주세요~";
	      
	      String result  = "";
	      
	      System.out.println(qMsg);
	      choice = sc.nextInt();
	      
	      if(choice ==1) {
	    	  result =aMsg;
	      }
	      else if(choice ==2) {
	    	  result =bMsg;
	      }
	      else if(choice ==3) {
	    	  result =oMsg;
	      }
	      else if(choice ==4) {
	    	  result =abMsg;
	      }
	      else {
	    	  result=errMsg;
	      }
```

만들었습니다. 원리는 같네요 ㅎㅎ 그런데 if 문마다 choice == ? 를 넣는게 슬슬 귀찮아지기 시작합니다. 그럴 때 필요한게 switch입니다.

**\-switch문**

switch(변수명){

case 값1:

실행할 문장;

case 값2:

실행할 문장;

case 값3:

실행할 문장;

default:

실행할 문장;

}

각 케이스에 변수명에서 가져온 값으로 그 케이스를 실행하면 됩니다. 이제 저것을 switch으로 해볼게요1

```
public static void main(String[] args) {
		String qMsg = "당신의 혈액형은?"
	            + "\n1. A"
	            + "\n2. B"
	            + "\n3. O"
	            + "\n4. AB";
	      
	      int choice = 0;
	      Scanner sc = new Scanner(System.in);
	      
	      String aMsg = "꼼꼼하고 세심하다.";
	      String bMsg = "추진력이 좋다.";
	      String oMsg = "사교성이 좋다.";
	      String abMsg = "착하다.";
	      String errMsg = "다시 시도해주세요~";
	      
	      String result  = "";
	      
	      System.out.println(qMsg);
	      choice = sc.nextInt();
	      
	      if(choice ==1) {
	    	  result =aMsg;
	      }
	      else if(choice ==2) {
	    	  result =bMsg;
	      }
	      else if(choice ==3) {
	    	  result =oMsg;
	      }
	      else if(choice ==4) {
	    	  result =abMsg;
	      }
	      else {
	    	  result=errMsg;
	      }

switch(choice){
case 1:
	result =aMsg;
	break;
case 2:
	result =bMsg;
	break;
case 3:
	result =oMsg;
	break;
case 4:
	result =abMsg;
	break;
default:
	result =errMsg;
	

}
	      System.out.println(result);

	}

}
```

아까와 초반에는 비슷한 코드를 썼지만 지금은 switch 문으로 더 단순하게 할 수 있을 것 같습니다. ^^

이번시간에는 여기까지 하겠습니다!