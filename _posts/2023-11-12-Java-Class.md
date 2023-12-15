---
title: [Java] 클래스 응용 문제
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


이번 시간에는 저번에 배웠던 클래스에 대해 복습을 하고 응용문제를 풀어보도록 하겠습니다.

**클래스(반)**

추상적인 개념으로서, 여러 개의 공통 요소를 선언해 놓은 공간입니다.

이 때 선언된 변수와 메소드를 필드라고 부르며, 이 필드에 접근하기 위해서는 구체적인 무언가가

필요합니다.

**객체화(instance)**

위에서 말한 필드에 접근하기 위해서 필요한 객체물입니다. 클래스라는 주제에서 하나의 구체적인 예를 든다고 생각하시면 됩니다.

따라서 객체화는 추상적인 개념을 구체화 시키는 작업입니다.

클래스명 객체명 = new 생성자();

**객체(instance variable)**

추상적인 개념을 구체화 시킨 것을 객체(instance variable)라고 합니다.

객체화를 통해 나온 객체도 instance variable 이지만, 클래스 필드 내에 선언된 변수도 instance variable 즉, 객체입니다. 좀 헷갈리지만 객체도 하나의 큰 변수의 개념이 들어가있다고 생각하시면 될 것 같습니다.

**생성자**

해당 클래스의 필드를 메모리에 할당한 후 할당된 필드의 주소값을 가지고 옵니다.

기본생성자가 아닌 직접생성자를 만들 경우, 초기화까지 진행할 수 있습니다.

**다형성**

오버 로딩(Overloading)

매개변수의 개수 또는 타입이 다르면 같은 이름의 메소드로 선언할 수 있습니다. 예를 들어,

```
 public SuperCar(String brand, String color, int price, String pw) {
      super();
      this.brand = brand;
      this.color = color;
      this.price = price;
      this.pw = pw;
   }

   public SuperCar(String brand, String color, int price) {
      super();
      this.brand = brand;
      this.color = color;
      this.price = price;
   }
```

이렇게 두개의 생성자가 있습니다. 하나는 매개변수가 4개이고 하나는 3개네요. 그렇다면 어떤 객체를 선언하고 직접생성자를 통해 선언 할때 4개의 매개변수를 한다면 첫번째 단락의 생성자로 사용하고, 3개면 그 밑에 있는 생성자를 사용합니다.

복습을 한번 해보았고요.. 이제 응용문제를 풀어보겠습니다. 정말 어려웠습니다.

자동차 시동에 관한 코딩인데요.

시동이 이미 켜져 있으면 경고 메세지 출력,

시동이 이미 꺼져 있으면 경고 메세지 출력

시동을 켤 때 비밀번호를 입력받은 후 일치 하면 켜집니다.

비밀번호를 3번 연속 잘못 입력한다면 "경찰 출동" 출력하기

클래스 단과 메인클래스를 구분해서 쓴 뒤 설명 하겠습니다.

```
class SuperCar {
   String brand;
   String color;
   int price;
   boolean check;
   String pw = "0000";
   int policeCnt;

   public SuperCar() {
      ;
   }

   public SuperCar(String brand, String color, int price, String pw) {
      super();
      this.brand = brand;
      this.color = color;
      this.price = price;
      this.pw = pw;
   }

   public SuperCar(String brand, String color, int price) {
      super();
      this.brand = brand;
      this.color = color;
      this.price = price;
   }
   // 비밀번호 검사
   boolean checkPw(String pw) {
      if (this.pw.equals(pw)) {
         return true;
      }
      return false;
   }

   // 시동 켜기
   boolean engineStart() {
      if (!check) {// 시동이 꺼졌다면 들어온다.
         check = true;// 시동 켜기
         return true;
      }
      return false;
   }

   // 시동 끄기
   boolean engineStop() {
      if (check) {
         check = false;
         return true;// 시동끄기 성공
      }
      return false;// 시동끄기 실패
   }
}
```

먼저 class SuperCar를 선언한뒤 필드로 brand, color, price, check(시동체크), pw(비밀번호), 그리고 policecnt를 선언 했습니다.

그 다음 기본생성자와 매개변수가 다른 생성자 2개를 선언하였습니다. 이제 메소드를 쭈욱 만들어 봅시다.

**비밀번호 검사**

boolean 타입의 메소드 checkPw를 만들고 매개변수는 문자열로 합니다. 만약 그 문자열이 SuperCar클래스의 pw와 같으면 ture값을

리턴합니다.

**시동켜기**

역시 boolean 타입의 메소드로 매개변수는 따로 없이 사용합니다. check가 false일 때, 즉 시동이 꺼져있을 때 if문이 실행되어서

시동을 켠상태로 두고 true값을 리턴합니다 조건이 성립이 안 된다면 false를 리턴합니다.

**시동끄기**

켜기와 반대로 만듭니다. 중요한 것은 시동이 꺼지는 메소드가 실행이 되었을때 켜기와 마찬가지로 true값을 리턴해야합니다.

시동끄기라는 메소드가 성공을 한거거든요..

이제 메인클래스로 가봅시다.

```
public class Shop {
   public static void main(String[] args) {
      
      SuperCar ferrari = new SuperCar();
      
      String msg = "1.시동켜기\n2.시동끄기";
      String pwMsg = "비밀번호 : ";
      int choice = 0;
      String pw = null;
      
      Scanner sc = new Scanner(System.in);
      
      while(true) {
         System.out.println(msg);
         choice = sc.nextInt();
         
         if(choice == 1) {
            if(!ferrari.check) {
               System.out.println(pwMsg);
               pw = sc.next();
               if(ferrari.checkPw(pw)) {
                  if(ferrari.engineStart()) {
                     System.out.println("시동 켜짐");
                     ferrari.policeCnt = 0;
                  }
               }else {
                  System.out.println("비밀번호 " + ++ferrari.policeCnt + "회 오류");
                  if(ferrari.policeCnt == 3) {System.out.println("경찰 출동중"); break;}
               }
            }else{
               System.out.println("이미 시동이 켜져 있습니다.");
            }
         }else if (choice == 2) {
            if(ferrari.engineStop()) {
               System.out.println("시동 꺼짐");
               break;
            }else {
               System.out.println("이미 시동이 꺼져 있습니다.");
            }
         }
      }
      
   }
}
```

먼저 객체를 하나 만들었습니다 이름은 ferrari 그리고 기본생성자로 사용하였습니다.

문자열 msg를 만든담에 "1.시동켜기\\n2.시동끄기" 를 대입시켰습니다.

역시 pwMsg = " 비밀번호 : "를 선언하고 초기화 했습니다. 그리고 위에 시동켜기와 시동끄기를 선택할 choice 변수도 만들었습니다.

그리고 비밀번호를 입력할 문자열 pw를 만들었습니다.

Scanner 클래스도 만들었구요. 이제 시작해봅시다. while문에 (true)를 넣어서 계속 반복으로 만들고 중간에 break를 넣어주겠습니다.

msg 를 출력하고 choice에 사용자가 입력을 합니다. choice가 1일때 , 즉 시동을 킬 때에는 먼저 ferrari.check로 시동이 꺼져 있는지 확인하고

꺼져있으면 본격적으로 시동켜기 메소드를 시작합니다. 비밀번호를 입력하고 ferrari.checkPw(pw)를 if 조건식으로 넣어서 true가 되면

그 다음으로 이동합니다. 그리고 시동켜기 메소드를 실행하고 드디어 "시동켜짐"을 출력합니다. 그리고 policeCnt를 0으로 다시 초기화 합니다.

초기화하지 않으면 평생 자동차 시동키면서 3번이라도 틀리면 경찰이 출동하기 때문이죠.

그리고 비밀번호가 틀릴경우, 비밀번호 오류 횟수를 출력하고 누적이 3회 되면 "경찰 출동"을 출력하고 break로 순환문을 빠져나옵니다.

그리고 위에서 시동이 켜져있다면 "이미 시동이 켜져있습니다"를 출력합니다.

그리고 choice 2로 넘어가봅시다.

이거는 크게 어려운게 없습니다. if문으로 시동끄기를 사용하고 break로 순환문을 빠져나옵니다. 시동끄기가 실패할경우 이미 시동이 켜져있다는

뜻이므로 "이미 시동이 꺼져 있습니다."를 출력합니다.

정말 복잡하죠? 저도 처음에 이해하려고 몇번을 봤는지 모르겠습니다. 하지만 이런거 하나하나 배워가면서 JAVA의 재미를 느껴서

정말 좋았습니다!!

다음시간에는 상속부분으로 돌아오겠습니다!