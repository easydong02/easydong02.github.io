---
title: 동기화(Synchronized) 파일입출력(FileIO)
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

이번시간에는 동기화와 파일입출력에 대해서 알아보려고 합니다.

**동기화**

멀티 쓰레드 환경에서 자원의 공유 문제를 해결하기 위해 공유하는 자원에 하나의 쓰레드만 접근할 수 있게 하는 것입니다. 저번 시간에 멀티쓰레드에 대해 알아봤을 때 우리는 new 연산자를 각각 쓰레드에 부여하여 각각의 자원을 갖게 하였는데요, 이번에는 하나의 자원(run()메소드)을 2개의 쓰레드에서 사용을 하고 거기에서 생기는 문제를 동기화로 해결해보겠습니다.

```
package sync;

public class ATM implements Runnable {

	int money =10000;

	@Override
	public void run() {
		for (int i = 0; i < 5; i++) {
			withdraw(1000);
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {;}
		}
	}
	public void withdraw(int money) {
			this.money -= money;
		System.out.println(Thread.currentThread().getName() + "이(가)"+ money +"원 출금");
		System.out.println("현재 잔액 : " + this.money + "원");
	}
}
```

Runnable 인터페이스를 지정한 ATM이라는 클래스를 만들었습니다. 초기 돈은 10000으로 설정하고 run()메소드를 재정의 했습니다. 그 전에 withdraw메소드를 보겠습니다. 인자로 들어온 수를 초기 돈 10000에서 빼준 뒤 출력문 2개를 작성했습니다. 그리고 다시 run()메소드로 가시면 저 withdraw메소드를 1000씩 인자를 주어 사용하고 이것을 5번 반복하도록 했습니다. sleep으로 속도를 느리게 하는 것도 하고요.

```
package sync;

public class Gs25 {

	public static void main(String[] args) {
		ATM atm = new ATM();
		
		Thread mom = new Thread(atm,"엄마");
		Thread son = new Thread(atm,"아들");
		
		mom.start();
		son.start();
	}

}
```

atm이라는 ATM 타입의 객체를 만들었습니다. 그리고 Thread 타입의 객체 mom과 son을 만들었는데요. 저번 시간의 멀티쓰레드와 다른 점은 둘다 Runnable Target자리에 객체 atm을 넣었다는 겁니다. 그리고 그 쓰레드의 이름은 "엄마", "아들"로 설정했습니다. 이제 이 두 쓰레드가 하나의 자원을 사용하겠네요. 10000에서 각각 1000씩 5번 반복하니까 결국 5번 반복뒤에는 0이 남겠네요. 한번 실행 해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/1.png)

?? 마지막 현재 잔잭이 1000이네요.. 심지어 계산도 뒤죽박죽입니다. 이것은 왜냐하면 하나의 자원에 복수의 쓰레드가 연결이 되어 있고 연산속도가 빠르다 보니 몇몇을 그냥 지나치고 연산이 되었기 때문입니다. 즉 동기화가 필요한 것이죠. 이제 한번 동기화를 해봅시다.

문법

원하는 코드에 synchronized() { } 의 바디 안에 넣거나 아니면 메소드 전체에 적용시키려면 해당 메소드 타입 앞에 synchronized를 적어주면 됩니다.

```
public synchronized void withdraw(int money) {
		
			this.money -= money;
		
		System.out.println(Thread.currentThread().getName() + "이(가)"+ money +"원 출금");
		System.out.println("현재 잔액 : " + this.money + "원");
	}
```

아까 그 withdraw메소드입니다. 달라진 것이 보이나요? 네 맞습니다. void 전에 synchronized가 보이시죠? 때문에 이 메소드는 여러 개의 쓰레드가 접근해도 하나의 쓰레드가 이 메소드를 다 사용하기 전까진 다른 메소드는 사용할 수 없는 것입니다. 그래서 오류를 방지하는 것이죠. 이제 다시 메인메소드가 있는 클래스로 가서 실행해보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/2.png)

잘보이시나요? 각각의 쓰레드가 차례차례 자원을 사용하는 모습을 볼 수 있습니다.

또 다른 방법으로는 코드에 동기화를 적용하는 것입니다. 코드를 드래그 하고 try ~ catch 할 때 처럼 Alt + Shift + z 를 누르면 저런 창이 나옵니다. 저기서 6번째 synchronized가 보이시죠? 그걸 누르면

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/3.png)

```
synchronized (this) {
				this.money -= money;
			}
```

이런 식으로 바디가 잡힙니다. 매개변수엔 this를 적읍시다.

이제 다른 건 몰라도 this.money -=money; 코드는 각각의 쓰레드가 차례대로 사용합니다. 따라서 출력문은 같이 사용하겠죠. 실행시켜보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/4.png)

이 상황에서는 코드에만 동기화를 적용했기 때문에 나머지 출력문은 동시에 나오네요. 그래도 연산에는 오류가 없으니 마지막 현재잔액에는 0원이 잘 출력되는 모습을 볼 수 있습니다. 신기하죠?

이제 좀 복잡한 응용문제를 풀어보겠습니다. 빵집인데요. 하루에 빵을 만드는 재료가 빵을 총 20개까지 만들 수 있습니다. 그리고 가게 선반에는 빵을 10개까지 놓을 수 있습니다. 이제는 빵을 만들고 10개가 되면 더이상 빵을 만들지 않게 할거고 손님이 와서 빵을 먹으면 선반에 자리가 남기 때문에 빵을 그때 또 만들겠습니다. 하지만 재료가 20개분이니 결국 총 20개를 만들겠죠? 일단 크게 빵을 만드는 것과 빵을 먹는 2개의 메소드를 만들고 시작해보죠.

```
package bakery;

public class BakeryPlate{
	
	private static BakeryPlate plate;
	
	public int breadCnt;
	public int eatCnt;
	
	
	private BakeryPlate() {;}
	
	public static BakeryPlate getInstance() {
		if(plate == null) {
			plate  =new BakeryPlate();
		}
		return plate;
	}
}
```

bakery패키지를 만들고 BakeryPlate클래스를 만들었습니다. 저는 여기서 어떤 것을 할거냐면 우리 예전에 Calendar클래스에 대해서 배운 것 기억하시나요? 여기서 Calendar클래스는 생성자를 한개밖에 못 만들었습니다. 따라서 여러가지 시간이 만들어지는 것을 용납하지 않았는데요, 이번에도 같은 방법을 써보겠습니다. 따라서 BakeryPlate 타입의 plate를 만들고 private과 static을 붙였습니다. 이러면 다른 클래스에서 BakeryPlate타입의 객체를 만들 수 없습니다. 그리고 breadCnt(빵의 개수)와 eatCnt(먹은 개수)를 전역변수로 선언하였습니다. 그리고 기본생성자를 만들었습니다. 그리고 그 밑에 getInstance()메소드를 만들었습니다. 여기에는 plate가 null값(초기값)이면 이 때 비로소 객체의 주소값을 만들고 넣어줍니다. null값이 아니라면 원래 만들어진 것을 계속 사용합니다. 따라서 이 타입의 객체는 한개만 있을 수 있겠죠?

```
	//빵 만들기
	public synchronized void makeBread() {
		if(breadCnt>9) {
			System.out.println("빵이 가득 찼습니다.");
			try {
				wait();//wait를 해야 else를 쓸 필요 없이 여기서 쓰레드가 멈춰버림 그리고 sync해줘야댐
			} catch (InterruptedException e) {;}
		}
		breadCnt++;
		System.out.println("빵을 1개 만들었습니다. 총: "+breadCnt+"개");
	}
```

빵만들기 메소드입니다. 선반에는 10개의 빵만 있을 수 있으니까 breadCnt가 10이 되면 더 이상 빵을 만들지 않아야 합니다. 따라서 wiat()메소드를 사용할건데요. 이 wait메소드를 만나면 그 메소드를 품은 메소드가 멈춥니다. 따라서 선반이 가득차면 빵을 안 만들죠. 그렇지 않다면 빵을 만들겠죠? 따라서 breadCnt를 1개 증가시키고 빵을 만들었다는 출력문을 만들었습니다. 그리고 동기화도 해주었습니다.

```
//빵 먹기
	public synchronized void eatBread() {
		if(eatCnt == 20) {
			System.out.println("빵이 다 떨어졌습니다.");
		}else if(breadCnt<1) {
			System.out.println("빵을 만들고 있습니다.");
		}else {
			breadCnt--;
			eatCnt++;
			System.out.println("빵을 1개 먹었습니다. 총: "+breadCnt+"개");
			notify();//wait() 메소드를 깨워주는 메소드 따로 try catch 안 해도 됨.
		}
	}
```

빵을 먹는 메소드입니다. 빵을 총 20개를 만든다고 했죠? 따라서 빵을 20개 먹었다면 재료가 없으니 빵이 다 떨어졌겠죠? 그에 따른 출력문을 만들었습니다. 그리고 breadCnt<1 라면 어떤 상황일까요? 선반에 있는 빵을 다 먹고 빵을 만들기 전에 또 먹으려는 거죠? 따라서 빵을 만들고 있다는 출력문을 썼습니다. 위의 둘다 조건에 해당되지 않는다면 정상적으로 빵을 먹는거겠죠. 그래서 breadCnt를 하나 줄이고 eatCnt를 하나 늘렸습니다. 그리고 빵을 먹고 선반에 남은 빵을 출력합니다. 선반에 빵을 먹었으니 빵을 또 만들어도 되겠죠? 그러니 아까 wait()로 멈춰버린 메소드를 다시 꺠웁니다. 여기에는 notify()메소드를 써주고 wait()메소드와 다르게 try~catch문을 안 써도 됩니다.

빵만들기와 빵먹기 메소드를 만들었으니 이제 멀티쓰레드에서 사용할 자원을 만들어봅시다.

```
package bakery;

public class BakeryMaker implements Runnable{
	boolean check;

	@Override
	public void run() {
		for (int i = 0; i <20; i++) {
			if(check) {break;}
			BakeryPlate.getInstance().makeBread();
			try {Thread.sleep(1000);} catch (InterruptedException e) {;}
		}
		if(!check) {
			System.out.println("재료가 소진되었습니다.");
		}
	}
}
```

BakeryMaker라는 클래스를 만들었습니다. boolean타입의 check를 만들었습니다. 초기값은 false겠죠? 그리고 run()메소드를 재정의합니다. 빵을 총 20개만들 수 있으니까 for문으로 20번반복을 설정했습니다. 이번에는 객체를 단 하나만 쓸 수 있게 설계를 했기 때문에 아까 만든 getInstance()메소드를 사용하고 makeBread()를 사용했습니다. 그리고 반복문에 넣은 이유는 반복을 해도 원래 객체가 만들어져 있다면 그 객체를 똑같이 반환하기 때문에 상관이 없기 때문입니다.

이제 메인 메소드를 사용하는 클래스를 만들겠습니다.

```
package bakery;

import java.util.Scanner;

public class Bakery2 {

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		BakeryMaker maker= new BakeryMaker();
		Thread makerThread = new Thread(maker);
		
		int choice =0;
		
		makerThread.start();
		
		while(true) {
		choice =sc.nextInt();
		if(choice==0) {
			BakeryPlate.getInstance().eatBread();
		}else  {
			//고객이 나가기를 눌렀다면, makerThread도 종료시킨다.
			maker.check=true;
			System.out.println("안녕히 가세요.");
			break;
		}
		}
	}
}
```

자원이 있는 BakeryMaker 타입의 객체 maker를 만든 뒤 Thread 타입의 makerThread를 만들어 여기에 maker 를 넣었습니다. 그리고 빵을 먹는 것과 나가기를 조작하기 위해 int 타입의 choice를 만들었습니다. 앞으로 0을 입력하면 빵을 먹는 거고 1을 입력하면 쓰레드를 종료시키고 나가기로 했습니다. 따라서 Scanner 타입 sc객체도 만들었습니다. 빵을 만드는 것은 반복을 굳이 주지 않아도 원래 자원에서 반복을 하니 그냥 start()메소드로 해주었습니다. 그리고 이제 우리가 조작을 할 건데, while 문을 만들어 일단은 무한반복으로 돌렸습니다. 그리고 choice에 sc.nextInt()로 우리가 정수를 입력하게 했습니다. 실행을 하게 되면 빵을 일단 계속 만들고 선반에 10개가 되면 wait()메소드가 발동 하겠죠?여기서 0을 입력하면 빵을 먹습니다. 그리고 notify로 깨워서 또 빵을 만들겠죠. 그리고 0이외의 숫자를 넣으면 아까 만든 check에 true를 주어 종료를 시킵니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/5.png)

잘 실행되죠? 저도 처음 배우고 만들 때 겁나 어려웠습니다. 하지만 찬찬히 코드를 풀어보고 따라가다보니까 할만하더라고요.. ㅎㅎ

**파일 입출력(JAVA Application 관점)**

**Writer(출력)**

BufferedWriter : 버퍼를 사용한 출력 클래스

FileWriter : 전달한 경로의 파일을 출력하기 위한 목적으로 열어줍니다.

전달한 경로에 파일이 없다면 새롭게 만든 후 열어줍니다.

File : 전달한 경로에 있는 파일의 정보를 담아줍니다.

전달한 경로에 파일의 유무 검사, 파일 삭제 등

**Reader(입력)**

BufferedReader : 버퍼를 사용한 입력 클래스

FileReader : 전달한 경로의 파일을 입력하기 위한 목적으로 열어줍니다.

전달한 경로에 파일이 없다면 FileNotFoundException이 발생합니다. 따라서 나중에 예외처리를 해주어야 합니다.

JAVA는 바로 운영체제와 소통하지 않습니다. 오직 JVM과 소통을 하고 그 JVM이 운영체제와 소통을 하죠. 그리고 .java파일과 예를 들어 text.txt파일과 입출력을 한다고 합시다. 그 둘 사이에 stream이라는 통로로 서로 byte를 주고 받는 것이죠. 그런데 그 byte들을 그냥 stream으로 보내버리면 어떤 문자열이 바로 나오는게 아니라 찔끔찔끔 출력이 됩니다. 그래서 우리한테 버퍼가 필요한 것이죠. 버퍼에는 두 파일의 중간에서 문자열 byte 를 받아와서 하나의 문자열이 완성되면 그것을 보냅니다. 그러면 중간에서 버퍼가 잡고있다가 완성한 문자열을 보내니까 딱 나오는 것이죠. 그래서 우리는 버퍼를 사용해서 입출력을 합니다. 출력부터 만들어봅시다.

```
BufferedWriter bw = new BufferedWriter(new FileWriter("test.txt"));
		bw.write("안녕!\n");
```

BufferedWriter 타입의 bw를 만들었습니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/6.png)

매개변수에는 Writer 타입이 나오는데요 이것은 위에 설명드린 FileWriter의 부모클래스 입니다. 그래서 업캐스팅으로 다시 FileWriter를 전달합니다. 그리고 또 여기의 매개변수에는 File타입을 적습니다. 그래서 우리는 텍스트 파일의 경로를 만들어서 출력을 할거니까 test.txt로 했습니다. 경로가 있다면 거기에 하고 없으면 만들어서 하겠죠? 그래서 BufferedWriter -> FileWriter -> File 타입으로 쭈욱 적었습니다.

그리고 이제 거기에 출력을 해야겠죠? 그 메소드가 write()메소드입니다. 따라서 bw.write("안녕!\\n)으로 안녕!과 줄바꿈을 넣었습니다. 실행시켜볼까요?콘솔창은 뜨지 않았지만 새로운게 생겼어요

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/7.png)

우리가 설정한 test.txt 파일이 생겼습니다. 들어가볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/8.png)

?! 우리는 분명 안녕! 이라고 적었는데 왜 안 보일까요? 왜냐하면 버퍼에 아직 남아있고 그 버퍼가 전달을 하지 않았습니다. 따라서 flush를 해주어야하는데요. 그것은 close()메소드입니다. 이거를 써서 버퍼에 있는 것을 test파일로 보냅니다.

```
BufferedWriter bw = new BufferedWriter(new FileWriter("test.txt"));
		bw.write("안녕!\n");
		bw.close();
```

이렇게 하고 다시 실행을 해보았습니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/9.png)

오 이제 드디어 들어갔습니다. ㅎㅎ 재밌네요. 우리는 이제 실행을 여러번 해서 안녕!을 여러번 출력하고 싶네요. 그런데 실행을 여러번 해도 똑같이 안녕! 한줄만 나오네요. 왜그럴까요? FileWriter()에 텍스트를 쓰면 여러번 실행해도 덮어쓰기가 되기 때문에 이어쓰기가 되지 않습니다. 그러면 다시 FileWriter를 봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/10.png)

우리는 그냥 "test.txt"를 했는데 이 메뉴에는 콤마 뒤에 boolean타입도 적을 수 있습니다. 여기에 true를 입력하면 append, 즉 이어쓸 수가 있습니다.

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/11.png)

잘 되죠? ㅎㅎ 재밌습니다

이제 출력했던 것을 다시 java로 입력하여 들여오겠습니다.

```
public static void main(String[] args) throws /*예외 던지기*/ IOException {
		BufferedReader br = null;
		try {
			br =new BufferedReader(new FileReader("test.txt"));
			String line = null;
			
			while((line = br.readLine())!= null) {
				System.out.println(line);// \n은 가져오지 않음.
			}
		} catch (FileNotFoundException e) {
			System.out.println("해당 경로에 파일이 없습니다."); 
		}
	}
```

메인 메소드에 throws 로 IOException을 적었습니다. 이것은 왜냐하면 외부에서 자료를 가져오면 이런 예외가 뜨기 때문입니다. 그래서 그때마다 try ~ catch로 하면 번거러우니까 메소드에 적어서 자동으로 사용자쪽으로 날리는 것입니다.

이제는 BufferedReader 타입의 객체를 만들고 null;값을 주었습니다. 아까 Writer에서는 파일의 경로가 없으면 만든다고 했죠? 그런데 Reader는 파일의 경로가 없으면 FileNotFoundException 예외가 생깁니다. 따라서 try ~ catch문으로 아까 그 예외를 처리했습니다. 그리고 객체 br에 아까와 비슷하지만 Reader로 우리가 썼던 test.txt를 가져옵니다. 그리고 String타입의 line을 만들고 null값을 주었습니다. while문에 조건식으로 readLine() !=null을 했습니다. readLine()은 그 다음 값이 있으면 그 값을 가져오고 없으면 null을 반환합니다. 따라서 그 다음 값이 없을 때까지 반복하는거죠. 그리고 출력문으로 그 line을 출력했습니다. 실행해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/12.png)

아까 작성했던 안녕!의 문자열들이 잘 실행이 되는 것을 볼 수 있었습니다.

이번시간에는 여기까지 하겠습니다!