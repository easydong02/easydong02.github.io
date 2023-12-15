---
title: [Java] 멀티쓰레드(MultiThread)
date: 2023-11-13 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
---

이번 시간에는 쓰레드에 대해서 알아보겠습니다. 배울 때 헷갈린 것도 있고 했지만 최대한 써보겠습니다.

쓰레드를 알아보기 전에 여러가지를 간단히 짚어보겠습니다.

**프로그램**

잘 짜여진 틀, 실행이 안 된 상태입니다.

**프로세스**

실행된 프로그램.

운영체제로부터 시스템 자원을 할당받는 작업의 단위입니다.

JAVA는 운영체제가 바로 실행시켜주지 않고 JVM에 의해 실행되기 때문에 JVM으로부터 시스템 자원을 할당받습니다.

우리가 작업관리자를 켰을 때 프로세스 목록에 여러가지 어플리케이션들이 백그라운드에서 실행되고 있는 것을 볼 수 있습니다.

**쓰레드**

프로세스 처리 경로, 전적으로 JVM에 의해 스케쥴링 됩니다.

**\- 단일 쓰레드**

처리 경로를 한 개만 가지고 이기 때문에 직렬적입니다. 동시에 많은 양을 처리하기 힘들기 때문에 상대적으로 비효율적이라고 할 수 있습니다.

하지만 하나의 작업에 문제가 발생하더라도 다른 작업에는 영향을 끼치지 않기 때문에 안정성이 보장된다는 장점이 있는데요.

심지어 설계도 멀티 쓰레드에 비해 쉽습니다.

**\- 멀티 쓰레드**

하나의 프로세스를 동시에 처리하는 것처럼 보이지만 사실은 매우 짧은 단위로 분할해서 차례로 처리하는 것입니다. 여러 개의 처리 경로를 가질 수 있도록 하며, 동시 작업이 가능해집니다. 설계하기 어려우며, 하나의 쓰레드 문제 발생 시 모든 쓰레드에 문제가 발생하게 됩니다. 그래도 JAVA 엡 개발 시 사용되는 웹 서버가 대표적인 멀티 쓰레드인데요. 왜냐하면 멀티 쓰레드로 설계했다면, 처리량 증가, 효율성 증가, 처리비용 감소의 장점이 있기 때문에

단점을 감수하고 설계하는 편입니다.

**멀티 쓰레드의 골칫덩어리 - 교착상태(DeadLock)**

멀티 쓰레드 중 쓰레드 간에 대기 상태가 종료되지 않아서 무한정 대기만 하는 비정상적인 상태를 가리킵니다.

교착상태인지를 판단했다면 전체 쓰레드를 깨워주거나, 하나의 쓰레드를 종료시켜주면 교착상태가 해결됩니다.

멀티 쓰레드를 구현하는 방법은 두 가지가 있습니다.

**1.Thread 상속**

첫 번째 방법은 직접 Thread클래스를 상속하는 방법입니다.

```
public class Thread1 extends Thread{
	private String data;
	
	public Thread1() {;}
	
	
	public Thread1(String data) {
		super();
		this.data = data;
	}

	@Override
	public void run() {
		for (int i = 0; i <10; i++) {
			System.out.println(data);
		}
	}	
}
```

먼저 Thread클래스를 상속받는 Thread1이라는 클래스를 만들었습니다.

String 타입의 변수 data를 만들어 주고 기본 생성자와, String data가 들어있는 초기화생성자를 만들었습니다.

그리고 중요한 게 run()이라는 메소드인데요. 이것이 바로 자원이라고 할 수 있는데요. 여기엔 소스코드가 와도 되고 메소드가 와도 됩니다. 중요한 것은 이것을 스케줄링하여 처리경로를 복수화할 수 있다는 것입니다. 이 때 우리가 말하는 멀티 쓰레드가 되는거죠. 여기에 data를 10번 출력하는 로직을 담았습니다.

이제 다른 클래스를 만들어보죠.

```
public class ThreadTest2 {

	public static void main(String[] args) {
		Thread t1 = new Thread1("홀");
		Thread t2 = new Thread1("짝");

		t1.start();
		t2.start();
	}

}
```

다른 클래스를 만들었습니다. Thread 타입의 객체 t1,t2를 만들었습니다. 그리고 Thread1클래스의 생성자를 만들고 각각 "홀", "짝"을 넣었습니다. 그리고 우리가 재정의 했던 run()메소드를 사용하면 될까요? 만약 t1.run(); t2.run();을하면 그냥 홀 10번, 짝10번 출력이 됩니다. 왜그럴까요? 왜냐하면 run() 자체가 멀티쓰레드가 되는 것이 아닌 이것을 스케줄링해야 멀티쓰레드가 구현이 되기 때문입니다. 그리고 그 run()이라는 자원을 스케줄링해주는 메소드가 바로 start()메소드입니다. 따라서 t1.start();로 하면 됩니다. 한번 해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/1.png)

나오긴 했는데.. 뭔가 우리가 생각한 느낌이 아니죠? 우리는 홀과 짝이 번갈아가면서 나오는 것을 상상했는데 말이죠..

왜그러냐면 시스템의 처리속도가 너무 빨라서 그냥 작업이 후루룩 지나갔기 때문입니다. 따라서 여기에 속도를 느리게해주어 우리가 원하는 결과를 만들고 싶은데요. 그럴 때 쓰는 메소드가 sleep()이라는 메소드입니다. 이것은 run()메소드를 재정의 할때 바디에 넣으시면 됩니다.

```
@Override
	public void run() {
		for (int i = 0; i <10; i++) {
			System.out.println(data);
			try {Thread.sleep(1000);} catch (InterruptedException e) {
				//A라는 쓰레드를 당장 종료시키고싶을 땐 강제로 하는 것보단 코드로서 자연스럽게 익셉션을 발생시키는게 좋다.
			}
		}
	}	
```

아까 만들었던 run()메소드에서 Thread.sleep(1000);를 넣었습니다. 이것은 1초의 딜레이를 주는 것인데요. 왜 여기에 try~catch문을 썼을까요?

그 이유는 바로 자연스러운 코드의 종료를 유도하기 위함입니다. sleep메소드를 try catch로 잡으면 자동적으로 예외타입이 InterruptedException로 설정이 됩니다. 2개의 쓰레드를 우리가 원하는 타이밍에 종료를 시켜주기 위해서 예외를 발생시키는 것이죠.

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/2.png)

이렇게 sleep()메소드를 사용하니까 1초씩 나오네요! 재밌습니다.

**2\. Runnable 인터페이스 지정**

두번째로는 Runnable이라는 인터페이스를 지정하는 방법입니다. JAVA특성상 다중 상소은 허용하지 않으므로 인터페이스를 지절하면 더 더 편하겠죠? 다만 우리가 쓰는 run()이나 start()메소드 등은 Thread클래스에 있으므로 한번더 객체화작업을 해야합니다. 이제 Runnable인터페이스를 지정한 Thread2를 만들어볼게요!

```
package day18;

public class Thread2 implements Runnable{

	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName());
			try {
				Thread.sleep(500);
			} catch (InterruptedException e) {;}
		}
	}
}
```

여기서도 run()메소드를 재정의했습니다. Thread클래스를 상속했을 때와 다른 것은 어떤 변수를 만들어서 값을 넣은 뒤 그 값을 출력했는데 여기에서는 currnetThread()메소드와 getName()메소드를 사용하겠습니다. 전자는 현재 그 쓰레드에 접근한 객체를 나타내는 메소드이고 후자는 그 접근한 객체의 이름을 반환하는 메소드입니다. 따라서 접근한 객체의 이름을 10번출력하겠네요. 뿐만 아니라 아까처럼 또 sleep으로 육안으로 멀티쓰레드가 작동하는 것을 보려고 합니다.

```
public class ThreadTest2 {

	public static void main(String[] args) {
		Thread2 t1 = new Thread2();
		Thread2 t2 = new Thread2();
		
		Thread thread1 = new Thread(t1,"홀");
		Thread thread2 = new Thread(t2,"짝");
		
		thread1.start();
		thread2.start();
	}

}
```

다른 클래스를 만들었습니다. 아까 만들었던 Thread2타입의 t1,t2를 만들었습니다. 하지만 t1객체로는 start()메소드를 사용할 수 없는데요, 왜냐하면 Thread2클래스는 Runnable인터페이스를 지정하고 Thread클래스를 상속하지 않았기 때문입니다. start()메소드는 Thread에만 있기 때문이죠. 따라서 Thread타입으로 다시 객체화를 해줍니다. 생성자를 보시면

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/3.png)

이런게 있습니다. 3번째를 보시면 Runnable target을 적는 부분이 보이시죠? 이것 때문에 우리는 Thread생성자에 Runnable을 지정한 객체를 넣을 수 있습니다. 그리고 5번째를 보시면 Runnable target 뿐만 아니라 String name도 보이시죠? 여기에서 그 객체의 쓰레드의 이름을 적는 것도 있습니다. 여기에 원하는 문자열을 적으면 우리는 Thread2클래스에서 재정의한 run()로 인해 그 이름이 나오겠습니다. 따라서 여기선 그 이름을 짝과 홀로 초기화를 해주었습니다.

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/4.png)

정상적으로 출력되는 모습을 볼 수 있습니다.

만약 이제 저 2개의 start()메소드가 끝나면 "메인쓰레드 종료"라는 문구를 출력하고 싶습니다. 그래서 마지막에

```
System.out.println("메인 쓰레드 종료.");
```

넣었습니다. 그렇다면 어떻게 될까요?

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/5.png)

저는 분명히 마지막에 넣었는데 왜 2번재 줄에 출력이 될까요? 내부적으로 정해진 순위가 있기 때문입니다. 그래서 원래는 저 문자열이 첫줄에 나오는데 홀짝을 출력하는 로직이 실행되는 중에 들어가있네요. 이럴 경우 치명적인 오류가 발생할 수 있습니다. 따라서 로직이 다 끝나기 전까지 "메인쓰레드 종료"라는 문자열이 실행이 안 되게 하고 싶습니다. 그럴 때 사용하는 메소드가 join()이라는 메소드입니다.

```
	Thread2 t1 = new Thread2();
		Thread2 t2 = new Thread2();
		
		Thread thread1 = new Thread(t1,"홀");
		Thread thread2 = new Thread(t2,"짝");
		
		thread1.start();
		thread2.start();
		
		try {thread1.join();
			thread2.join();	} catch (InterruptedException e) {;}
		
		System.out.println("메인 쓰레드 종료.");
```

join메소드를 써봤습니다. sleep()메소드처럼 try catch문으로 예외처리를 해줍니다. 이제 저 println()메소드가 실행되려면 thread1, threade2의 start()메소드가 전부 끝나야 합니다.

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/6.png)

이제 마지막에 정상적으로 출력되는 것이 보입니다.

**응용문제**

문제를 풀어봅시다.

동물원에 동물 3마리가 있습니다. 각 동물은 울음 소리가 다르고 2마리의 동물은 동시에 웁니다.

나머지 1마리 동물은 2마리 동물이 모두 울고 나서 마지막에 웁니다. 편의상 동물은 자기만의 소리로 10번씩 운다고 합시다.

여기서는 Runnable 인터페이스를 지정하는 방식으로 멀티쓰레드를 구현하겠습니다.

```
package test;

public class Animal implements Runnable{
	
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread().getName());
			try {
				Thread.sleep(300);
			} catch (InterruptedException e) {;}
		}
		
	}

}
```

먼저 Animal이라는 클래스를 만든 뒤 Runnable인터페이스를 지정합니다.

그리고 run()메소드를 재정의해주었습니다.

```
String[] sounds = {"어흥", "음메", "냐옹"};
				Thread[] arThread = new Thread[sounds.length];
				
				for (int i = 0; i < arThread.length; i++) {
					arThread[i] = new Thread(new Animal(), sounds[i]);
				}
				
				//join()은 이미 시작된 쓰레드를 멈출 수는 없다.
				for (int i = 0; i < arThread.length; i++) {
					arThread[i].start();
					if(i != 0) {
						try {	arThread[i].join();	} catch (InterruptedException e) {;}
					}
				}
```

이것은 AnimalTest라는 클래스의 메인메소드의 바디입니다.

String타입의 배열 sounds를 만든 뒤 각각 "어흥", "음메", "냐옹"을 넣었습니다. 그리고 Thread타입의 배열 arThread를 만든뒤 그 칸의 개수는 sounds의 길이만큼으 로 만들었습니다. 이제 for문으로 arThread배열에 생성자를 하나씩 넣습니다. 여기엔 Runnable를 지정한 Animal의 생성자주소값과 그 옆에는 그 쓰레드의 이름을 적는 부분에 아까 만든 sounds의 원소를 하나씩 넣습니다.

그리고 for문으로 start()하나씩 하고 싶은데요, 헷갈리시죠? 저 for문을 풀어보겠습니다.

```
				arThread[0].start();
				arThread[1].start();
				try {
					arThread[1].join();
				} catch (InterruptedException e) {;}
				arThread[2].start();
				try {
					arThread[2].join();
				} catch (InterruptedException e) {;}
```

풀어보았습니다. 처음 join문을 실행하기 전에

arThread\[0\].start();

arThread\[1\].start();

를 실행합니다. 그런데 arThread\[0\]는 join()메소드를 쓰지 않았는데 어떻게 멈추지 않았을까요? join()메소드는 위에 적혀진것, 즉 먼저 실행되버린 로직은 멈추질 못합니다. 그 밑에 있는것들만 멈추죠. 따라서 \[0\]과 \[1\]의 start()메소드는 계속 작동을 하다가 이제 \[2\]가 마지막에 시작이 되는거죠.

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/7.png)

냐옹만 어흥과 음메가 실행이 다 되고 실행되는 모습을 볼 수 있습니다.

이번시간은 여기서 마칩니다 ^~^ 재밌었습니다!