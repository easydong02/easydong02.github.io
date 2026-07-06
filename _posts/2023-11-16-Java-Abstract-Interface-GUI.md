---
title: "[Java] 추상클래스 인터페이스 JAVA GUI"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **추상 클래스·인터페이스**를 다시 정리하고(다중 상속 관점), **AWT를 이용한 GUI**를 만들며 **JavaDoc을 100% 활용하는 법**까지 익힌다.

| 개념 | 핵심 |
|------|------|
| **추상 클래스** | 추상 메소드를 포함, 상속 시 구현 강제 (`extends`) |
| **인터페이스** | **추상 메소드만** 있는 순수 틀 (`implements`, 다중 가능) |

---

## 1. 추상 클래스 — 강제되는 틀

```java
// 이 클래스는 의도적으로 불완전하게 정의 (추상)
public abstract class MusicPlayer {
    public abstract void supportMp3();   // 바디 없음 → 상속받는 쪽이 구현
    public abstract void volumeUP();
    public abstract void volumeDown();
}
```

> 💡 미래에 어떤 기기가 만들어질지 예측할 수 없으므로, 구체 내용은 비워두고 **각 기기(자식 클래스)가 알아서 구현**하도록 강제한다.

---

## 2. 인터페이스 — 다중 상속의 타협점

블루투스 스피커를 만들되, **날기도(Drone) 하고 타기도(Roller)** 하는 요상한 물건으로 만들고 싶다. 이때 인터페이스가 필요하다.

```java
public interface Drone {
    public void fly();        // 인터페이스는 abstract 생략 가능
}
public interface Roller {
    public abstract void run();
}
```

```java
public class BSpeaker extends MusicPlayer implements Roller, Drone {
    //                 └── 상속(1개) ──┘  └── 구현(여러 개) ──┘
    public void connect()     { System.out.println("근거리 블루투스 연결"); }
    public void supportMp3()  { System.out.println("블루투스 MP3 로드"); }
    public void volumeUP()    { System.out.println("볼륨 올리기"); }
    public void volumeDown()  { System.out.println("볼륨 내리기"); }
    public void run()         { System.out.println("달린다."); }
    public void fly()         { System.out.println("난다."); }
}
```

> 💡 **왜 Drone·Roller를 인터페이스로 만들었나?** 자바는 클래스 **다중 상속을 금지**한다. 하지만 현실을 반영하려면 다중 상속이 필요하다. 그래서 타협으로, **상속은 하나(`extends`)만 하되 인터페이스는 여러 개(`implements`)** 구현할 수 있게 했다.

### 업 캐스팅

```java
BSpeaker bs = new BSpeaker();
bs.supportMp3(); bs.fly(); bs.run();

Drone d = bs;   // 스피커는 Drone 형이다 (업 캐스팅)
d.fly();
Roller r = bs;  // 스피커는 Roller 형이다
r.run();
```

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/1.png)

`BSpeaker`는 `MusicPlayer`이자 `Drone`이자 `Roller`이므로, 세 타입 모두로 업 캐스팅할 수 있다.

---

## 3. AWT로 GUI 만들기

자바의 GUI 객체들은 **`java.awt` 패키지**에 있다.

```java
package com.koreait.gui;
import java.awt.*;   // awt 하위 전체 import

class GuiTest1 {
    public static void main(String[] args) {
        Frame frame = new Frame();     // 윈도우 역할
        frame.setVisible(true);        // 기본은 안 보임 → 보이게
        frame.setSize(300, 400);

        FlowLayout flow = new FlowLayout();
        frame.setLayout(flow);         // 내부 배치 방법

        Button bt = new Button("나 버튼");
        TextField t_id = new TextField(20);
        Checkbox ch1 = new Checkbox("딸기", true);
        Choice ch = new Choice();      // select 박스
        TextArea area = new TextArea(7, 20);

        ch.add("이메일 선택"); ch.add("google.com");
        ch.add("naver.com");  ch.add("daum.net");

        frame.add(bt);   frame.add(t_id); frame.add(ch1);
        frame.add(ch);   frame.add(area);
    }
}
```

주요 컴포넌트를 정리하면 이렇다.

| 컴포넌트 | 역할 |
|------|------|
| `Frame` | 윈도우 창 (기본은 안 보임 → `setVisible(true)`) |
| `Button` | 버튼 |
| `TextField` | 한 줄 입력 |
| `TextArea` | 여러 줄 입력 |
| `Checkbox` | 체크박스 |
| `Choice` | 드롭다운(select) |

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/8.png)

---

## 4. JavaDoc 활용법

`Frame`이 정확히 뭔지 모를 때, **[JavaDoc](https://docs.oracle.com/javase/8/docs/)**을 찾아본다.

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/2.png)

| JavaDoc에서 확인하는 것 | 내용 |
|------|------|
| **상속 계층** | 어떤 부모를 상속받았는지 (최상위는 `Object`) |
| **Summary** | 생성자·필드·메소드 목록 |
| **상속 메소드** | 부모·조부모로부터 물려받은 메소드 |

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/6.png)

> 💡 `setVisible()`은 `Frame`이 아니라 조상인 `Window` 클래스에 있다. JavaDoc의 "상속 메소드" 섹션에서 이를 찾아, `boolean` 매개변수를 `true`로 주면 창이 보인다는 것을 알 수 있다. **`Object`는 모든 클래스의 최상위 부모**로, 마치 헌법 같은 존재다.

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/7.png)

---

## 📝 정리

```
추상 클래스·인터페이스·GUI
├─ 추상 클래스  추상 메소드 포함, extends로 상속(1개)
├─ 인터페이스   추상 메소드만, implements로 다중 구현
├─ 다중상속     클래스는 1개 상속 + 인터페이스 여러 개로 타협
├─ AWT GUI      Frame + Button/TextField/Checkbox... + add()
└─ JavaDoc      상속 계층·메소드 조회로 모르는 클래스 활용
```

| 개념 | 한 줄 정의 |
|------|------|
| **추상 클래스 vs 인터페이스** | 상속(1개) vs 다중 구현 |
| **업 캐스팅** | 구현한 인터페이스 타입으로도 담기 |
| **AWT** | 자바의 GUI 패키지 (`java.awt`) |
| **JavaDoc** | 클래스의 API를 조회하는 문서 |

인터페이스는 자바가 **다중 상속을 우회**하는 방법이다. 그리고 모르는 클래스를 만나도 **JavaDoc으로 상속 계층과 메소드를 조회**하면 얼마든지 활용할 수 있다는 점이 이번 실습의 핵심이다.
