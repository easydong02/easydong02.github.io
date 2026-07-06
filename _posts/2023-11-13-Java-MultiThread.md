---
title: "[Java] 멀티쓰레드(MultiThread)"
date: 2023-11-13 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **쓰레드(Thread)**를 다룬다. 배울 때 헷갈리는 개념이지만, 프로그램 → 프로세스 → 쓰레드로 이어지는 흐름과 멀티쓰레드 구현 두 가지 방법을 차근차근 정리한다.

---

## 1. 프로그램 · 프로세스 · 쓰레드

| 용어 | 정의 |
|------|------|
| **프로그램** | 잘 짜여진 틀, **실행 안 된 상태** |
| **프로세스** | **실행된 프로그램**, 시스템 자원을 할당받는 작업 단위 |
| **쓰레드** | 프로세스의 **처리 경로**, JVM이 스케줄링 |

> 💡 자바는 OS가 직접 실행하지 않고 **JVM에 의해** 실행되므로, JVM으로부터 시스템 자원을 할당받는다.

---

## 2. 단일 쓰레드 vs 멀티 쓰레드

| 구분 | 단일 쓰레드 | 멀티 쓰레드 |
|------|------|------|
| 처리 경로 | 한 개 (직렬) | 여러 개 (동시 처리처럼) |
| 효율 | 상대적으로 낮음 | **처리량↑ 효율↑ 비용↓** |
| 안정성 | **한 작업 문제가 다른 작업에 영향 없음** | 한 쓰레드 문제가 전체에 영향 |
| 설계 | 쉬움 | 어려움 |

> 💡 멀티 쓰레드는 사실 동시가 아니라 **아주 짧은 단위로 분할해 차례로** 처리하는 것이다. 단점(설계 난이도·전파성)에도 불구하고 웹 서버 등은 처리량·효율 때문에 멀티 쓰레드로 설계한다.

> ⚠️ **교착상태(DeadLock)**: 쓰레드 간 대기 상태가 끝나지 않아 무한정 대기하는 비정상 상태. 전체 쓰레드를 깨우거나 하나를 종료시켜 해결한다.

---

## 3. 구현 방법 ① — Thread 상속

`Thread` 클래스를 상속하고 **`run()`을 재정의**한다. `run()`이 바로 스케줄링되는 '자원'이다.

```java
public class Thread1 extends Thread {
    private String data;
    public Thread1() {}
    public Thread1(String data) { super(); this.data = data; }

    @Override
    public void run() {                    // 스케줄링될 자원
        for (int i = 0; i < 10; i++) {
            System.out.println(data);
        }
    }
}
```

```java
Thread t1 = new Thread1("홀");
Thread t2 = new Thread1("짝");
t1.start();   // run()이 아니라 start()!
t2.start();
```

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/1.png)

> ⚠️ **`run()`이 아니라 `start()`를 호출**해야 한다. `t1.run()`을 직접 부르면 그냥 순차 실행(홀 10번 → 짝 10번)이 된다. `run()`을 **스케줄링**해 멀티쓰레드로 만드는 메소드가 **`start()`**다.

### sleep() — 눈으로 확인하기

처리 속도가 너무 빨라 결과가 뒤섞인다. `sleep()`으로 딜레이를 준다.

```java
@Override
public void run() {
    for (int i = 0; i < 10; i++) {
        System.out.println(data);
        try {
            Thread.sleep(1000);   // 1초 딜레이
        } catch (InterruptedException e) {
            // 쓰레드를 자연스럽게 종료 유도
        }
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/2.png)

> 💡 `sleep()`을 `try~catch`로 감싸면 예외 타입이 자동으로 **`InterruptedException`**이 된다. 강제 종료보다 코드로 자연스럽게 예외를 발생시켜 쓰레드를 원하는 타이밍에 종료하기 위함이다.

---

## 4. 구현 방법 ② — Runnable 인터페이스

자바는 다중 상속을 지원하지 않으므로, **`Runnable` 인터페이스를 구현**하면 더 유연하다. 단, `run()`/`start()`는 `Thread`에 있으므로 **한 번 더 Thread로 객체화**해야 한다.

```java
public class Thread2 implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName());  // 접근한 쓰레드 이름
            try { Thread.sleep(500); } catch (InterruptedException e) {}
        }
    }
}
```

```java
Thread2 t1 = new Thread2();
Thread2 t2 = new Thread2();

// Runnable 객체를 Thread 생성자에 넣고, 이름도 지정
Thread thread1 = new Thread(t1, "홀");
Thread thread2 = new Thread(t2, "짝");

thread1.start();
thread2.start();
```

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/4.png)

> 💡 `Thread(Runnable target, String name)` 생성자를 이용해, Runnable 객체와 쓰레드 이름을 함께 넘긴다. `getName()`이 이 이름("홀"/"짝")을 반환한다.

### join() — 쓰레드 종료를 기다리기

```java
thread1.start();
thread2.start();
try {
    thread1.join();   // thread1, 2가 끝날 때까지
    thread2.join();   // 아래 코드를 기다리게 함
} catch (InterruptedException e) {}
System.out.println("메인 쓰레드 종료.");
```

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/6.png)

> ⚠️ `join()` 없이 마지막에 출력문을 두면, "메인 쓰레드 종료"가 **로직 도중에** 끼어 출력된다(내부 스케줄링 순위 때문). `join()`은 대상 쓰레드가 끝날 때까지 다음 코드를 **대기**시킨다.

| 메소드 | 역할 |
|------|------|
| `start()` | run()을 스케줄링해 쓰레드 시작 |
| `sleep(ms)` | 지정 시간만큼 일시정지 |
| `join()` | 대상 쓰레드가 끝날 때까지 대기 |

---

## 5. 응용 문제 — 동물원

**문제**: 동물 3마리 중 2마리는 동시에 울고, 나머지 1마리는 앞 둘이 끝난 뒤 마지막에 운다. (각 10번씩)

```java
public class Animal implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(Thread.currentThread().getName());
            try { Thread.sleep(300); } catch (InterruptedException e) {}
        }
    }
}
```

```java
String[] sounds = {"어흥", "음메", "냐옹"};
Thread[] arThread = new Thread[sounds.length];

for (int i = 0; i < arThread.length; i++) {
    arThread[i] = new Thread(new Animal(), sounds[i]);
}

for (int i = 0; i < arThread.length; i++) {
    arThread[i].start();
    if (i != 0) {   // 0번은 join 안 함
        try { arThread[i].join(); } catch (InterruptedException e) {}
    }
}
```

풀어 쓰면 이렇게 동작한다.

```java
arThread[0].start();   // 어흥 시작
arThread[1].start();   // 음메 시작 (0,1 동시 진행)
arThread[1].join();    // 음메가 끝날 때까지 대기
arThread[2].start();   // 냐옹은 앞 둘이 끝난 뒤 시작
arThread[2].join();
```

![Desktop View](/assets/img/Programming-Language/Java/MultiThread/7.png)

> 💡 `arThread[0]`은 `join`을 안 걸었는데 왜 멈추지 않았을까? **`join()`은 이미 시작된(위에 적힌) 쓰레드는 멈추지 못하고, 그 아래 코드만 대기시킨다.** 그래서 0·1은 계속 돌다가, 2(냐옹)가 마지막에 시작된다.

---

## 📝 정리

```
멀티쓰레드
├─ 개념     프로그램 → 프로세스(실행) → 쓰레드(처리 경로)
├─ 구현①    Thread 상속 + run() 재정의 → start()
├─ 구현②    Runnable 구현 → new Thread(runnable, name).start()
├─ sleep()  딜레이 (InterruptedException)
└─ join()   대상 종료까지 이후 코드 대기 (시작된 것은 못 멈춤)
```

| 개념 | 한 줄 정의 |
|------|------|
| **쓰레드** | 프로세스의 처리 경로 |
| **start()** | run()을 스케줄링해 실행 |
| **join()** | 쓰레드가 끝날 때까지 대기 |
| **DeadLock** | 서로 무한 대기하는 비정상 상태 |

멀티쓰레드의 핵심은 **`run()`을 직접 부르지 않고 `start()`로 스케줄링**한다는 것, 그리고 **`join()`으로 실행 순서를 제어**한다는 것이다. `join`이 이미 시작된 쓰레드는 못 멈춘다는 미묘한 동작까지 이해하면 응용 문제도 풀린다.
