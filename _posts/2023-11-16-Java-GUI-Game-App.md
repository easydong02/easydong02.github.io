---
title: "[Java] Java GUI로 게임 만들기 1"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 먼저 **컬렉션 프레임워크(Set·List·Map)**를 정리하고, 이어서 자바 **그래픽스(Graphics)**로 간단한 **슈팅 게임**(주인공 1기 + 적군 5기)을 만든다.

> **컬렉션 프레임워크란?** 객체를 모아서 처리하는 데 유용한 객체들을 지원하는 라이브러리.

| 계열 | 특징 |
|------|------|
| **Set** | 순서 없음, 중복 없음 (유무 검사에 빠름) |
| **List** | 순서 있음, 배열과 유사하되 크기 유동적 |
| **Map** | Key-Value 쌍 (key로 value 조회) |

---

## 1. Set — 순서 없는 집합

인터페이스라 직접 객체를 못 만들고 `HashSet`으로 사용한다. 순서가 없어 **`Iterator`로 순서를 부여**해 순회한다.

```java
Set set = new HashSet();
set.add("사과"); set.add("오렌지"); set.add("딸기");
set.add("포도"); set.add("수박");

Iterator it = set.iterator();       // 순서 부여
while (it.hasNext()) {
    String e = (String) it.next();  // 요소 꺼내기
    System.out.println(e);
}
```

> 💡 `Iterator`는 JDK 5 이전의 `Enumeration`을 대체한다. `hasNext()`로 다음 값 유무, `next()`로 값을 꺼낸다.

---

## 2. List — 순서 있는 집합 (ArrayList)

배열과 유사하지만 **크기가 유동적**이고, **객체만** 담을 수 있다.

```java
List<String> list = new ArrayList<String>();   // <String> = 제네릭 (타입 제한)
list.add("바나나"); list.add("멜론"); list.add("파인애플");

// 인덱스 접근
for (int i = 0; i < list.size(); i++) System.out.println(list.get(i));

// 향상된 for문 (JDK 5+)
for (String obj : list) System.out.println(obj);
```

| 배열 | ArrayList |
|------|------|
| 크기 고정 | **크기 유동적** (add로 자동 확장) |
| 기본형·객체형 | **객체형만** |

---

## 3. Map — Key-Value 쌍 (HashMap)

열쇠(key)와 자물쇠(value)처럼 쌍으로 저장되어, **key로 value를 조회**한다.

```java
Map<String, String> map = new HashMap<String, String>();
map.put("name1", "Smith");
map.put("name2", "Scott");

System.out.println(map.get("name1"));   // Smith

Set<String> keySet = map.keySet();       // key만 Set으로
Iterator<String> it = keySet.iterator();
while (it.hasNext()) {
    System.out.println(map.get(it.next()));
}
```

---

## 4. 그래픽스 — 현실 vs 전산

게임 그래픽을 이해하려면 현실의 그림 그리기에 빗대면 쉽다.

| 현실 | 전산 |
|------|------|
| 화가 / 도화지 | **UI 컴포넌트** |
| 팔레트(색) | **Graphics 객체** (색·도형·문자·이미지) |
| 붓(그리는 행위) | **paint() 메소드** |

> 💡 지금까지는 이미 그려진 것을 사용만 했다면, 이제 **`paint()`를 재정의**해 그리기에 직접 개입한다. `paint()`는 "그리는 행위"일 뿐, 실제 효과는 팔레트인 **Graphics 객체**가 전담한다.

**이미지 로드**에는 두 객체를 쓴다.

| 객체 | 역할 |
|------|------|
| `Toolkit` | 로컬 파일을 디렉터리 형태로 불러옴 |
| `Image` | 불러온 이미지를 담음 |

---

## 5. 게임 구현 (4개 클래스)

```
AnimationApp (JFrame, 실행부)
   └─ GamePanel (JPanel, paint·키 입력)
        ├─ Hero  (주인공)
        └─ Enemy (적군 x5)
```

### AnimationApp — 실행부

```java
public class AnimationApp extends JFrame {
    GamePanel gamepanel;
    public AnimationApp() {
        gamepanel = new GamePanel();
        add(gamepanel);
        this.pack();                        // 패널 크기만큼 프레임 맞춤
        this.setLocationRelativeTo(null);   // 화면 정중앙
        setVisible(true);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
    }
    public static void main(String[] args) { new AnimationApp(); }
}
```

### GamePanel — 이미지 로드 & 배치

```java
public class GamePanel extends JPanel implements KeyListener {
    public static final int WIDTH = 1280, HEIGHT = 640;
    Toolkit kit = Toolkit.getDefaultToolkit();   // 인터페이스 → getDefault로
    Image bg_img, hero_img;
    Hero hero;
    Image[] enemy_image = new Image[5];
    Enemy[] enemy = new Enemy[5];

    public GamePanel() {
        this.setPreferredSize(new Dimension(WIDTH, HEIGHT));
        bg_img   = kit.getImage("...\\bg.jpg");       // 이미지 메모리에 로드
        hero_img = kit.getImage("...\\plane.png");
        for (int i = 0; i < 5; i++)
            enemy_image[i] = kit.getImage("...\\e" + (i+1) + ".png");

        hero = new Hero(hero_img, 100, 150);          // 주인공 생성
        for (int i = 0; i < 5; i++)                   // 적군 5기 생성
            enemy[i] = new Enemy(enemy_image[i], WIDTH - 100, 100 * (i+1));

        addKeyListener(this);
        this.setFocusable(true);   // 포커스를 프레임→패널로 이동
    }
    // ... paint, keyPressed 아래
}
```

> 💡 이미지는 스택처럼 **나중에 그린 것이 위에** 온다. `setFocusable(true)`는 실행 시 프레임에 있던 포커스를 **패널로 옮겨** 키 입력을 받게 한다.

### paint() 재정의 & 키 입력

```java
@Override
public void paint(Graphics g) {
    g.drawImage(bg_img, 0, 0, WIDTH, HEIGHT, this);   // 배경
    hero.render(g);                                    // 주인공
    for (int i = 0; i < enemy.length; i++)
        enemy[i].render(g);                            // 적군
}

@Override
public void keyPressed(KeyEvent e) {
    if      (e.getKeyCode() == e.VK_RIGHT) hero.x += 10;
    else if (e.getKeyCode() == e.VK_LEFT)  hero.x -= 10;
    else if (e.getKeyCode() == e.VK_UP)    hero.y -= 10;
    else if (e.getKeyCode() == e.VK_DOWN)  hero.y += 10;
    repaint();   // 다시 그리기 → 이동 반영
}
```

> 💡 `drawImage`의 마지막 인자 `this`는 **이 메소드를 감시할 주체(ImageObserver)**를 지정한다. 방향키를 누르면 `hero.x/y`를 바꾸고 **`repaint()`**로 다시 그려 이동을 반영한다.

### Hero / Enemy — 개체 클래스

```java
public class Hero {
    Image hero_img;
    int x, y;
    public Hero(Image hero_img, int x, int y) {
        this.hero_img = hero_img; this.x = x; this.y = y;
    }
    public void tick() {}   // 물리 변화(좌표 변경)용
    public void render(Graphics g) {   // 화면에 그리기
        g.drawImage(hero_img, x, y, 120, 75, null);
    }
}
```

> 💡 `Enemy` 클래스는 `Hero`와 사실상 동일하다(이미지·좌표·render). 게임 개체를 **객체로 분리**해 각자 독립적인 상태(위치·이미지)를 갖게 한 것이 포인트다.

![Desktop View](/assets/img/Programming-Language/Java/GUI-Game-App/1.png)

주인공 1기와 적군 5기가 화면에 나오고, 방향키로 주인공을 움직일 수 있다.

---

## 📝 정리

```
컬렉션 & 게임 그래픽스
├─ Set     순서X 중복X (HashSet + Iterator)
├─ List    순서O 유동적 (ArrayList, 제네릭)
├─ Map     Key-Value (HashMap, key로 조회)
├─ 그래픽   paint() 재정의 + Graphics(팔레트) + Toolkit/Image
└─ 게임     JFrame→JPanel, Hero/Enemy 객체, 키입력→repaint()
```

| 개념 | 한 줄 정의 |
|------|------|
| **Set/List/Map** | 순서X / 순서O / 키-값 |
| **paint(Graphics)** | 화면 그리기(재정의로 개입) |
| **Toolkit/Image** | 이미지 로드 |
| **repaint()** | 상태 변경 후 다시 그리기 |

컬렉션은 "객체를 모아 다루는 표준 도구"이고, 게임 그래픽의 핵심은 **`paint()` 재정의 + 개체를 객체로 분리 + 입력 후 `repaint()`**다. 다음 편에서 이 게임에 움직임과 충돌을 더한다.
