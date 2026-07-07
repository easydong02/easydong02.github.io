---
title: "[Java] Java GUI로 게임 만들기 2"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

저번의 간단한 게임을 이번엔 제법 그럴듯하게 발전시킨다. **7개 클래스**로 나뉘며, **추상 클래스 상속·게임 쓰레드(루프)·어댑터·익명 클래스**같은 중요한 개념이 총출동한다.

```
GameMain (JFrame, 메뉴바)
   └─ GamePanel (JPanel, 게임 쓰레드 + paintComponent)
        └─ GameComponents (추상, 공통 필드/메소드)
             ├─ GameBg   (배경)
             ├─ Hero     (주인공)
             ├─ Enemy    (적군)
             └─ Bullet   (총알)
```

---

## 1. GameMain — 메뉴바 + 익명 클래스 리스너

메뉴바로 게임 시작/일시정지/종료를 제어한다.

```java
public class GameMain extends JFrame {
    JMenuBar bar;
    JMenu menu;
    JMenuItem[] item = new JMenuItem[3];
    GamePanel gamePanel;

    public GameMain() {
        bar = new JMenuBar();
        menu = new JMenu("게임설정");
        item[0] = new JMenuItem("게임시작");
        item[1] = new JMenuItem("Pause");
        item[2] = new JMenuItem("게임종료");
        gamePanel = new GamePanel();

        for (JMenuItem obj : item) menu.add(obj);
        bar.add(menu);
        setJMenuBar(bar);
        add(gamePanel);
        pack(); setVisible(true);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);

        // 익명 클래스로 리스너 즉석 구현
        item[0].addActionListener(new ActionListener() {
            @Override public void actionPerformed(ActionEvent e) { gamePanel.check = true; }
        });
        item[1].addActionListener(new ActionListener() {
            @Override public void actionPerformed(ActionEvent e) { gamePanel.check = false; }
        });
        item[2].addActionListener(new ActionListener() {
            @Override public void actionPerformed(ActionEvent e) { System.exit(0); }
        });
    }
    public static void main(String[] args) { new GameMain(); }
}
```

> 💡 리스너를 `implements`하지 않고 `new ActionListener() { ... }`로 만든 게 **익명 클래스**다. **재사용하지 않고 일시적으로** 쓸 때 유용하다. 이름이 없어 다시 참조할 수 없다.

> ⚠️ **메인 쓰레드**의 역할은 프로그램 운영(GUI 이벤트 감지, 그래픽 처리)이다. 절대 대기 상태나 무한 루프에 빠지면 안 된다. (스마트폰 개발에선 메인 쓰레드의 루프 자체가 에러다.)

---

## 2. GameComponents — 공통을 모은 추상 클래스

배경·주인공·적군·총알은 좌표·크기·속도·이미지가 공통이다. 중복을 없애려 **추상 클래스**로 묶는다.

```java
public abstract class GameComponents {
    int x, y, width, height, velX, velY;   // 공통 필드
    Image img;

    public GameComponents(int x, int y, int width, int height, int velX, int velY, Image img) {
        super();
        this.x = x; this.y = y; this.width = width; this.height = height;
        this.velX = velX; this.velY = velY; this.img = img;
    }
    public abstract void tick();              // 운동량 변경 (자식마다 구현)
    public abstract void render(Graphics g);  // 화면에 그리기
}
```

| 메소드 | 역할 |
|------|------|
| `tick()` | 물리값(좌표) 변경 |
| `render(g)` | 변경된 상태를 화면에 그림 |

이제 모든 게임 요소가 이를 상속해 **자기만의 tick/render**를 구현한다. 예를 들어 배경은 계속 왼쪽으로 흐르다 끝에 닿으면 되돌아온다.

```java
public class GameBg extends GameComponents {
    public GameBg(int x, int y, int w, int h, int vx, int vy, Image img) {
        super(x, y, w, h, vx, vy, img);
    }
    public void tick() {
        this.x += this.velX;
        if (this.x < -width) this.x = width;   // 끝에 닿으면 되돌리기 (무한 스크롤)
    }
    public void render(Graphics g) { g.drawImage(img, x, y, width, height, null); }
}
```

> 💡 `Hero`, `Enemy`, `Bullet`도 구조가 거의 같다. 공통을 부모(추상)에 몰아넣어 **중복을 최소화**한 것이 이 설계의 핵심이다.

---

## 3. GamePanel — 게임의 심장(쓰레드 + 루프)

### 쓰레드란?

> 실행 중인 프로그램이 **프로세스**이고, 그 안에서 **독립적으로 동시에 실행되는 세부 단위**가 **쓰레드**다. OS가 프로세스를 관리하듯, 자바 쓰레드는 **JVM(소형 OS)에 맡겨야** 동시 실행된다.

### 생성자 — 게임 쓰레드 + 어댑터

```java
public GamePanel() {
    setPreferredSize(new Dimension(WIDTH, HEIGHT));
    setFocusable(true);
    createBg(); createHero(); createEnemy();   // 생성은 메소드로 분리

    gameThread = new Thread() {
        @Override
        public void run() {
            while (true) {
                if (check) gameLoop();          // check가 true일 때만 루프
                try { Thread.sleep(10); }        // FPS 조절 (1000분의 10초)
                catch (InterruptedException e) {}
            }
        }
    };
    gameThread.start();   // JVM에 맡김

    // 어댑터로 필요한 키 이벤트만 구현
    this.addKeyListener(new KeyAdapter() {
        public void keyPressed(KeyEvent e) {
            switch (e.getKeyCode()) {
                case KeyEvent.VK_LEFT:  hero.velX = -5; break;
                case KeyEvent.VK_RIGHT: hero.velX =  5; break;
                case KeyEvent.VK_UP:    hero.velY = -5; break;
                case KeyEvent.VK_DOWN:  hero.velY =  5; break;
                case KeyEvent.VK_SPACE: createBullet();  break;
            }
        }
        public void keyReleased(KeyEvent e) {
            switch (e.getKeyCode()) {   // 키 떼면 정지
                case KeyEvent.VK_LEFT: case KeyEvent.VK_RIGHT: hero.velX = 0; break;
                case KeyEvent.VK_UP:   case KeyEvent.VK_DOWN:  hero.velY = 0; break;
            }
        }
    });
}
```

> 💡 **게임 루프**: `gameLoop()`은 내부적으로 `repaint()`만 호출한다. 쓰레드가 이걸 계속 반복하면 모든 요소의 좌표 변경이 다시 그려진다 — **멈추지 않는 심장**이다. Pause 메뉴가 `check=false`로 만들면 루프가 멈춘다.

> 💡 **어댑터**: `KeyListener`를 직접 구현하면 3개 메소드를 다 구현해야 하지만, `KeyAdapter`(모두 빈 구현)를 상속하면 **필요한 것만** 재정의할 수 있다. `Thread.sleep(10)`은 CPU가 너무 빨라 게임이 미친 듯 빨라지는 걸 막는 **FPS 조절**이다.

### 이미지 로드 (classpath)

```java
public void createBullet() {
    try {
        BufferedImage buffImg = ImageIO.read(
            this.getClass().getClassLoader().getResource("ball.png"));
        bulletArray.add(new Bullet(hero.x + 100, hero.y + 50, 20, 20, 15, 0, buffImg));
    } catch (IOException e) { e.printStackTrace(); }
}
```

> 💡 로컬 디렉터리 경로 대신 **`getClassLoader().getResource()`**로 이미지를 **클래스패스**에서 가져온다. 총알은 스페이스바를 누를 때마다 생기므로 **`ArrayList<Bullet>`**에 누적한다.

### paintComponent — 모든 요소 그리기

```java
@Override  // repaint() → update() → paint()의 업그레이드판
public void paintComponent(Graphics g) {
    g.setColor(Color.YELLOW);
    g.fillRect(0, 0, WIDTH, HEIGHT);

    gameBg[0].tick(); gameBg[0].render(g);
    gameBg[1].tick(); gameBg[1].render(g);
    hero.tick();      hero.render(g);

    for (Bullet b : bulletArray) { b.tick(); b.render(g); }   // 누적 총알
    for (Enemy en : enemy)       { en.tick(); en.render(g); }
}
public void gameLoop() { repaint(); }   // 심장
```

![Desktop View](/assets/img/Programming-Language/Java/GUI-Game-App-2/1.png)

주인공·적군·배경이 움직이고, 스페이스바로 총알도 날아간다.

---

## 📝 정리

```
슈팅 게임 (심화)
├─ 익명 클래스  메뉴 리스너를 일회용으로 구현
├─ 추상 클래스  GameComponents로 공통(좌표·속도·render) 통합
├─ 게임 쓰레드  while + sleep(FPS) + gameLoop(repaint) = 심장
├─ 어댑터       KeyAdapter로 필요한 키 이벤트만 구현
└─ 이미지       getClassLoader().getResource()로 classpath 로드
```

| 개념 | 한 줄 정의 |
|------|------|
| **추상 클래스 상속** | 공통 필드/동작을 부모에 몰아 중복 제거 |
| **게임 루프** | 쓰레드가 repaint를 반복 (FPS는 sleep) |
| **어댑터** | 필요한 리스너 메소드만 재정의 |
| **익명 클래스** | 일회용 리스너 구현 |

이번 게임은 **"공통은 추상 클래스로, 반복 렌더링은 쓰레드 루프로, 입력은 어댑터로"**라는 세 축이 맞물린다. 반드시 반복해서 보고 스스로 만들 수 있을 때까지 내 것으로 만들어야 할 예제다.
