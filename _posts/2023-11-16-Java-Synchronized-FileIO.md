---
title: "[Java] 동기화(Synchronized) 파일입출력(FileIO)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 멀티쓰레드에서 **하나의 자원을 여러 쓰레드가 공유**할 때 생기는 문제를 **동기화(synchronized)**로 해결하고, 이어서 **버퍼 기반 파일 입출력**을 다룬다.

> **동기화란?**
> 멀티쓰레드 환경에서 자원 공유 문제를 해결하기 위해, **공유 자원에 하나의 쓰레드만 접근**하게 하는 것.

---

## 1. 동기화 문제 상황 — ATM

`ATM` 하나(자원)를 엄마·아들 두 쓰레드가 공유해서 출금한다.

```java
public class ATM implements Runnable {
    int money = 10000;
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            withdraw(1000);
            try { Thread.sleep(500); } catch (InterruptedException e) {}
        }
    }
    public void withdraw(int money) {
        this.money -= money;
        System.out.println(Thread.currentThread().getName() + "이(가) " + money + "원 출금");
        System.out.println("현재 잔액 : " + this.money + "원");
    }
}
```

```java
ATM atm = new ATM();
Thread mom = new Thread(atm, "엄마");   // 같은 atm 공유!
Thread son = new Thread(atm, "아들");
mom.start();
son.start();
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/1.png)

> ⚠️ 10000에서 각각 1000씩 5번, 총 10번이면 잔액 0이 되어야 하는데 **1000이 남고 계산도 뒤죽박죽**이다. 하나의 자원에 여러 쓰레드가 연결되고 연산이 빨라, 중간 과정이 **씹히며** 계산됐기 때문이다. → **동기화 필요**

---

## 2. 동기화 적용

| 방법 | 문법 |
|------|------|
| **메소드 전체** | `public synchronized void 메소드()` |
| **특정 코드 블록** | `synchronized (this) { ... }` |

### 메소드 동기화

```java
public synchronized void withdraw(int money) {   // synchronized 추가
    this.money -= money;
    System.out.println(...);
}
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/2.png)

> 💡 이제 한 쓰레드가 이 메소드를 **다 쓰기 전까지 다른 쓰레드는 접근 못 한다.** 그래서 차례차례 실행되어 오류가 사라진다.

### 블록 동기화

```java
synchronized (this) {
    this.money -= money;   // 이 코드만 순차 접근
}
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/4.png)

> 💡 블록만 동기화하면 **핵심 연산(`money -= money`)만 순차**로 처리되고 출력문은 동시에 나온다. 그래도 연산에는 오류가 없어 최종 잔액 0이 정확히 나온다.

---

## 3. 응용 — 빵집 (wait / notify + 싱글톤)

**조건**: 하루 재료로 빵 20개 생산 가능, 선반엔 10개까지. 선반이 차면 생산 중단, 손님이 먹으면 다시 생산.

### 싱글톤 — 선반은 하나뿐

`Calendar`처럼 **객체를 하나만** 만들도록 `private` 생성자 + `getInstance()`를 쓴다.

```java
public class BakeryPlate {
    private static BakeryPlate plate;   // 유일한 인스턴스
    public int breadCnt;
    public int eatCnt;

    private BakeryPlate() {}             // private 생성자 (외부 생성 차단)

    public static BakeryPlate getInstance() {
        if (plate == null) plate = new BakeryPlate();   // 없을 때만 생성
        return plate;                                    // 항상 같은 객체 반환
    }
}
```

### wait / notify

```java
// 빵 만들기
public synchronized void makeBread() {
    if (breadCnt > 9) {
        System.out.println("빵이 가득 찼습니다.");
        try { wait(); } catch (InterruptedException e) {}   // 여기서 멈춤
    }
    breadCnt++;
    System.out.println("빵을 1개 만들었습니다. 총: " + breadCnt + "개");
}

// 빵 먹기
public synchronized void eatBread() {
    if (eatCnt == 20)      System.out.println("빵이 다 떨어졌습니다.");
    else if (breadCnt < 1) System.out.println("빵을 만들고 있습니다.");
    else {
        breadCnt--; eatCnt++;
        System.out.println("빵을 1개 먹었습니다. 총: " + breadCnt + "개");
        notify();          // wait()로 멈춘 쓰레드를 깨움
    }
}
```

| 메소드 | 역할 |
|------|------|
| `wait()` | 현재 쓰레드를 **멈춤** (try-catch 필요) |
| `notify()` | `wait()`로 멈춘 쓰레드를 **깨움** (try-catch 불필요) |

> 💡 선반이 차면 `wait()`로 생산을 멈추고, 손님이 먹으면 `notify()`로 생산을 재개한다. 이것이 전형적인 **생산자-소비자** 패턴이다.

```java
// 소비: 0 입력=먹기, 그 외=종료
while (true) {
    choice = sc.nextInt();
    if (choice == 0) BakeryPlate.getInstance().eatBread();
    else { maker.check = true; System.out.println("안녕히 가세요."); break; }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/5.png)

---

## 4. 파일 입출력 — 버퍼

> 자바는 OS와 직접 소통하지 않고 **JVM을 거친다.** 파일 사이는 **스트림**이라는 통로로 byte를 주고받는데, 그냥 보내면 찔끔찔끔 출력된다. 그래서 **버퍼**가 중간에서 문자열을 모아 완성되면 한 번에 보낸다.

| 방향 | 클래스 | 역할 |
|------|------|------|
| 출력 | `BufferedWriter` | 버퍼 사용 출력 |
| | `FileWriter` | 경로의 파일 열기(**없으면 생성**) |
| 입력 | `BufferedReader` | 버퍼 사용 입력 |
| | `FileReader` | 경로의 파일 열기(**없으면 예외**) |

### 출력 — flush(close) 잊지 말기

```java
BufferedWriter bw = new BufferedWriter(new FileWriter("test.txt"));
bw.write("안녕!\n");
bw.close();   // ★ 버퍼 내용을 파일로 보냄(flush)
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/9.png)

> ⚠️ `close()`가 없으면 내용이 **버퍼에 남아** 파일이 비어 보인다. `close()`가 버퍼를 flush해 파일에 기록한다.

> 💡 `FileWriter("test.txt")`는 실행할 때마다 **덮어쓰기**된다. 이어쓰려면 두 번째 인자에 **`true`**(append)를 준다: `new FileWriter("test.txt", true)`.

### 입력

```java
public static void main(String[] args) throws IOException {   // 예외 던지기
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader("test.txt"));
        String line = null;
        while ((line = br.readLine()) != null) {   // 다음 줄이 없을 때까지
            System.out.println(line);              // \n은 안 가져옴
        }
    } catch (FileNotFoundException e) {
        System.out.println("해당 경로에 파일이 없습니다.");
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Synchronized-FileIO/12.png)

> 💡 `readLine()`은 다음 줄이 있으면 그 값을, 없으면 **null**을 반환한다. 그래서 `while((line = readLine()) != null)`이 전체 읽기에 딱 맞는다. Reader는 파일이 없으면 `FileNotFoundException`이 나므로 예외 처리가 필요하다.

---

## 📝 정리

```
동기화 & 파일 입출력
├─ 동기화     synchronized (메소드/블록)로 자원 순차 접근
├─ wait/notify  멈춤/깨움 (생산자-소비자)
├─ 싱글톤     private 생성자 + getInstance()로 인스턴스 1개
├─ 출력       BufferedWriter + close()(flush), append는 true
└─ 입력       BufferedReader.readLine()이 null일 때까지
```

| 개념 | 한 줄 정의 |
|------|------|
| **synchronized** | 공유 자원에 하나의 쓰레드만 접근 |
| **wait / notify** | 쓰레드를 멈추고 깨우기 |
| **싱글톤** | 인스턴스를 하나만 유지 |
| **버퍼(close)** | 모아서 보내기, close로 flush |

동기화는 "공유 자원을 한 번에 하나씩"이라는 원칙이고, 파일 출력은 **`close()`(flush)를 잊지 않는 것**이 핵심이다. 처음엔 어려웠지만 코드를 찬찬히 따라가니 할 만했다.
