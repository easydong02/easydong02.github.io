---
title: [Java] Java GUI로 게임 만들기 2
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
오늘은 엄청 힘든 시간이었습니다. 보통 어떤 것을 배우면 좀 익히고 실전에 사용하는데 이번에는 실전에 사용하면서 하나씩 배우는 느낌이어서..

이번엔 저번에 간단한 게임을 만들었는데 이번엔 꽤 그럴듯하게 만들었습니다! 무려 7개의 클래스가 있는데요.. 또 엄청 중요한 개념들이 안에 포함하고 있답니다..

먼저 게임의 실행부인 GameMain부터 해보겠습니다.

//GameMain

```
package com.koreait.project0807.game;

import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

import javax.swing.JFrame;
import javax.swing.JMenu;
import javax.swing.JMenuBar;
import javax.swing.JMenuItem;

public class GameMain extends JFrame{
	JMenuBar bar;
	JMenu menu;
	JMenuItem[] item= new JMenuItem[3];
	GamePanel gamePanel;
	
	public GameMain() {
		
		//생성
		bar =new JMenuBar();
		menu = new JMenu("게임설정");
		item[0] = new JMenuItem("게임시작");
		item[1] = new JMenuItem("Pause");
		item[2] = new JMenuItem("게임종료");
		
		
		
		gamePanel = new GamePanel();
		
		//조립
		for(JMenuItem obj : item) {
			menu.add(obj);
		}
		bar.add(menu);
		setJMenuBar(bar);
		add(gamePanel);
		pack();
		setVisible(true);
		setLocationRelativeTo(null);//화면 가운데로 위치
		setDefaultCloseOperation(EXIT_ON_CLOSE);
		
	
		//이벤트 연결
		item[0].addActionListener(new ActionListener() {//게임 시작
			
			@Override
			public void actionPerformed(ActionEvent e) {
				gamePanel.check=true;
			}
		});
		item[1].addActionListener(new ActionListener() {//게임 잠시 멈춤
			
			@Override
			public void actionPerformed(ActionEvent e) {
				gamePanel.check=false;
			}
		});
		item[2].addActionListener(new ActionListener() {//프로그램 종료
			
			@Override
			public void actionPerformed(ActionEvent e) {
				System.exit(0); //프로세스 종료
			}
		});
	}
	
	public static void main(String[] args) {
		//지금까지 자바의 메인 실행부라 불리는 녀석은.. 메인스레드였다..
		//메인스레드, 즉 메인실행부의 역할은 프로그램 운영, 특히 gui에서는 이벤트 감지,
		//그래픽 처리(그리는 역할)
		//따라서 절대로 대기상태나, 무한루프에 빠지게 해서는 안 된다!
		//참고로 스마트폰 개발에서는 메인 쓰레드의 루프는 그 자체가 에러사항이다..
		
		new GameMain();
		
	}
}
```

메인메서드가 존재하는 GameMain입니다. 먼저 우리는 메뉴바를 만들어 게임을 일시정지,재개,프로그램 종료를 메뉴바에 아이템으로 넣어서 전체 프로그램을 조절해보겠습니다.

그래서 J메뉴바,J메뉴,그리고 아이템은 복수니까 배열로 선언했습니다. 그리고 가장 중요하고 뒤에 설명할 GamePanel 클래스를 부착시켰습니다.

그리고 각각 메뉴아이템한테 이름을 주었습니다. 그리고 pack()으로 윈도우 창을 패널과 같게하고 조립과 윈도우 설정을 하던대로 했습니다! 그리고 아까 아이템들을 액션리스너와 연결시켜 실질적으로 아이템을 눌렀을 때 각각의 기능이 실행될 수 있도록 하였습니다. 그런데 여기서 특이한건 리스너를 implements하지 않았는데 쓰고있네요 ? 다시 한번 볼까요?

```
		item[0].addActionListener(new ActionListener() {//게임 시작
			
			@Override
			public void actionPerformed(ActionEvent e) {
				gamePanel.check=true;
			}
		});
```

보통 우리는 이 클래스에서 리스너를 사용하기 위하여 implements를 써서 리스너와 연결을했는데요. 이것을 보면 new ActionListener()라고 되어있네요? 여기까진 이해가 잘 가죠? 객체를 따로 안 만들고 하는 기법인데요, 따라서 어디서 정의하지도 않았으니 이 매개변수 안에서 선언을 해버립니다.. 신기하죠? 그런데 여기서 new로 만들어지고 추상메서드를 재정의까지 했는데.. 이 클래스의 이름은 무엇일까요? 없습니다 ㅎㅎ 그래서 이것을 익명클래스라고 합니다. 언제 많이쓰이냐면 재사용하지 않을 경우, 일시적으로 사용해야할 때 씁니다. 왜냐하면 그냥 매개변수 안에 일시적으로 만든것이기 때문이죠. 이름도 없어서 다시 저것을 쓸 수도 없습니다. 지금도 이 아이템을 누를 때 말고는 사용할 일이 없으니 그냥 저렇게 일회용으로 만든것이죠

이제 다른 클래스를 살펴보죠. 저번 시간에 주인공이랑 적군들이랑 클래스를 만들 때 거의 비슷한 부분이 있었죠? 이번엔 날아가는 총알도만들고 할거라 최대한 중복인 부분은 제하고 싶습니다. 그래서 우리가 배운게 뭐죠? 틀이죠. 그중 가장 공통되는 것들을 모아서 하나의 추상클래스로 만들었습니다. 앞으로 주인공이나 적군 등등 이 게임의 안에 등장하는 모든 요소를 이 클래스를 상속하게 했습니다. 보시죠.

//GameComponents

```
package com.koreait.project0807.game;

import java.awt.Graphics;
import java.awt.Image;

//이 객체는 게임에 등장하는 모든 ~~~객체들의 최상위 객체이며, 
//보편적인 기능 및 코드를 보유할 예정

public abstract class GameComponents {

	int x;
	int y;
	int width;
	int height;
	int velX;
	int velY;
	Image img;

	public GameComponents(int x, int y, int width, int height, int velX, int velY, Image img) {
		super();
		this.x = x;
		this.y = y;
		this.width = width;
		this.height = height;
		this.velX = velX;
		this.velY = velY;
		this.img = img;
	}

	// 미래를 알 수 없다.. 어떤 자식이 어떤행동을 할지..
	public abstract void tick();

	public abstract void render(Graphics g);

}
```

추상클래스 GameComponents를 만들었습니다. 대부분 공통적으로 들어가는 요소가 좌표와, 크기, 그리고 움직여야하니까 속도를 넣고, 각각 이미지가 있을테니까 그것들을 추상클래스의 멤버변수로 구현했습니다. 그리고 그 변수들로 생성자도 만들었습니다. 그리고 앞으로 쓸 tick()메서드(운동량 지정)와 render()메서드(실제로 이미지를 패널에 띄울)를 추상메서드로 지정했습니다. 앞으로 이 게임 안에 들어가는 요소중 상속을 받으면 이것들을 강제적으로 각각의 개성으로 구현해야합니다.

//GameBg (배경화면)

```
package com.koreait.project0807.game;

import java.awt.Graphics;
import java.awt.Image;

//게임 배경을 제어하기 위해 객체로 정의한다..
public class GameBg extends GameComponents{

	
	
	public GameBg(int x, int y , int width,int height,int velX,int velY,Image img) {
		super(x,y,width,height,velX,velY,img);
	
	}
	
	public void tick() {//물리값 변화시킬 메서드
		this.x=this.x + this.velX;
		this.y=this.y + this.velY;
		if(this.x <-width) {
			this.x=width;
		}
		
	}
	public void render(Graphics g) {//tick에 의해 변화된 물리값을 화면에 적용할(그래픽처리) 렌더링 메서드
		g.drawImage(img, x, y, width, height, null);//옵저버는 비워놓기
	}
}
```

저는 배경화면을 고정으로 하고 싶지 않았습니다. 보통 게임하면 앞으로 가는 듯한 느낌을 주기 위해 배경을 뒤쪽으로 이동시키는 모습을 자주 볼 수 있을것입니다. 그리고 그 배경을 2개 만들어서 붙인다음에 처음에 나온배경이 끝까지 뒤로가면 다시 앞으로 불러와서 계속 배경을 돌려막기(?)하겠습니다! tick()메서드를 보시면 x좌표값에 가로축이동 속도인 velX에 중첩적용되는 것을 볼 수 있습니다. 그리고 그 변한 위치를 다시 그려주는 render()메서드를 사용하였습니다.

//Hero 주인공 요소

```
package com.koreait.project0807.game;

import java.awt.Graphics;
import java.awt.Image;
import java.awt.Rectangle;

//주인공을 정의한다
public class Hero extends GameComponents {

	
	public Hero(int x, int y, int width, int height, int velX, int velY, Image img) {
		super(x,y,width,height,velX,velY,img);

	}

	//부모꺼를 나한테 맞게 최적화 시켜야함, 즉 오버라이딩 하자!
	public void tick() {
		this.x += this.velX;
		this.y += this.velY;
	}
	
	public void render(Graphics g) {
		g.drawImage(img, x, y, width,height , null);
	}
}
```

주인공 클래스입니다. extends GameComponents를 하니까 당연히 생성자와 메서드들을 구현해야합니다. 그래서 주인공의 개성을 넣어주었습니다.

//Enemy 적군 클래스

```
package com.koreait.project0807.game;

import java.awt.Graphics;
import java.awt.Image;

public class Enemy extends GameComponents{

	public Enemy(int x, int y, int width, int height, int velX, int velY, Image img) {
		super(x, y, width, height, velX, velY, img);
	}

	@Override
	public void tick() {
		this.x +=this.velX;
		this.y +=this.velY;
		
	}

	@Override
	public void render(Graphics g) {
		g.drawImage(img, x, y, width, height, null);
	}

}
```

적군 클래스입니다. 뭐 없죠? ㅎㅎ 주인공과 비슷합니다. 이것도 하나의 틀이라고 할 수있죠 나중에 다 조립할때 값을 정해줄겁니다!

//Bullet 클래스 총알입니다!

```
package com.koreait.project0807.game;

import java.awt.Graphics;
import java.awt.Image;

//총알을 정의한다
public class Bullet extends GameComponents{

	public Bullet(int x, int y, int width, int height, int velX, int velY, Image img) {
		super(x, y, width, height, velX, velY, img);
	}

	@Override
	public void tick() {
		this.x +=this.velX;
		this.y +=this.velY;
	}

	@Override
	public void render(Graphics g) {
		g.drawImage(img, x, y, width, height,null);
	}

}
```

마찬가지 입니다. 이제 하이라이트인 GamePanel클래스를 만들어서 여기서 완성을 시켜주겠습니다.

```
package com.koreait.project0807.game;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import java.awt.event.KeyAdapter;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.net.URL;
import java.util.ArrayList;

import javax.imageio.ImageIO;
import javax.swing.JPanel;

public class GamePanel extends JPanel {
	public static final int WIDTH = 1280;
	public static final int HEIGHT = 640;
	Thread gameThread;// 게임 운영요 쓰레드 정의
	boolean check = true;
	GameBg[] gameBg = new GameBg[2];
	Hero hero ;
	Enemy[] enemy = new Enemy[5];
	Bullet bullet;
	ArrayList<Bullet> bulletArray= new ArrayList<Bullet>();
	
	
	public GamePanel() {
		setPreferredSize(new Dimension(WIDTH, HEIGHT));
		setFocusable(true);
		
		createBg();//배경생성
		createHero();//주인공생성
		createEnemy();//적군생성
		
		gameThread = new Thread() {
			@Override
			public void run() {
				while (true) {
					if (check) {
						gameLoop();
					}
					try {
						Thread.sleep(10);// 게임의 속도 조절 FPS 개념
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		};
		gameThread.start();
		//패널과 리스너와의 연결 : 어댑터라는 리스너..
		this.addKeyListener(new KeyAdapter() {
			public void keyPressed(KeyEvent e) {
				int key=e.getKeyCode();
				
				switch(key) {
					
				case KeyEvent.VK_LEFT :{hero.velX=-5;break;}
				case KeyEvent.VK_RIGHT :{hero.velX=5;break;}
				case KeyEvent.VK_UP:{hero.velY=-5;break;}
				case KeyEvent.VK_DOWN :{hero.velY=5;break;}
				case KeyEvent.VK_SPACE :{createBullet();break;}
				
				
				}
			}
			public void keyReleased(KeyEvent e) {
				int key=e.getKeyCode();
				
				switch(key) {
					
				case KeyEvent.VK_LEFT :{hero.velX=0;break;}
				case KeyEvent.VK_RIGHT :{hero.velX=0;break;}
				case KeyEvent.VK_UP:{hero.velY=0;break;}
				case KeyEvent.VK_DOWN :{hero.velY=0;break;}
				
				
				}
			}
		});
	}
	
	
	

	// 배경을 생성
	public void createBg() {
//		Class myClass=this.getClass();

		URL url = this.getClass().getClassLoader().getResource("game_bg.jpg");
		try {// 에러가 날 가능성이 있는 코드..
			BufferedImage buffImg= ImageIO.read(url);
			gameBg[0] = new GameBg(0, 0, 1280, 640, -1, 0, buffImg);
			gameBg[1] = new GameBg(1280, 0, 1281, 640, -1, 0, buffImg);
		} catch (IOException e) {// 혹여나 에러가 나면 비정상적으로 종료하지말고, catch문영역을 수행해..
			e.printStackTrace();
		}


	}
	
	//주인공을 생성
	public void createHero() {
		URL url = this.getClass().getClassLoader().getResource("plane.png");
		try {
			BufferedImage buffImg= ImageIO.read(url);
			hero =new Hero(100,200,120,75,0,0,buffImg);
		} catch (IOException e) {
			e.printStackTrace();
		}

	}
	//적군을 생성
	public void createEnemy() {
		String[] imgName = {"e1.png","e2.png","e3.png","e4.png","e5.png"};
		
		try {
			for (int i = 0; i < imgName.length; i++) {
				BufferedImage buffImg = ImageIO.read(this.getClass().getClassLoader().getResource(imgName[i]));
				enemy[i] = new Enemy(WIDTH,100+(100*i),90,90,-2,0,buffImg);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}

		
	}

	//총알을 생성
	public void createBullet() {
		try {
			BufferedImage buffImg = ImageIO.read(this.getClass().getClassLoader().getResource("ball.png"));
			 bulletArray.add(new Bullet(hero.x+100,hero.y+50,20,20,15,0,buffImg));
		} catch (IOException e) {
			e.printStackTrace();
		}

		
	}
	
	@Override
	//repaint() --> update() --> paint();
	public void paintComponent(Graphics g) {
		g.setColor(Color.YELLOW);
		g.fillRect(0, 0, WIDTH, HEIGHT);
		
		gameBg[0].tick();
		gameBg[0].render(g);
		gameBg[1].tick();
		gameBg[1].render(g);
		
		hero.tick();
		hero.render(g);
		
		//누적된 총알 모두의 tick(),render()호출
		for (int i = 0; i < bulletArray.size(); i++) {
			Bullet bullet =bulletArray.get(i); //요소 꺼내기
			bullet.tick();
			bullet.render(g);
		}
		
		
		for(int i=0;i<enemy.length;i++) {
			enemy[i].tick();
			enemy[i].render(g);
			
		}
	}

	// 게임의 심장역할을 해줄, 게임루프 정의..
	public void gameLoop() {
		// System.out.println("gameLoop() 호출..");
		repaint();
	}

	//인터페이스 구현시, 재정의할 메서드 3개이상 될때, 사용하지도 않는 메서드를 클래스 내에 방치하는 것은
	//올바르지 못하다. 해결책)3개이상의 구현메서드가 있을 때는 sun에서 개발자 대신 인터페이스를 구현해놓은
	//클래스를 지원하는데, 이 클래스를 가리켜 Adapter(어댑터)라 한다..
}
```

대미를 장식할 GamePanel입니다. 먼저 클래스를 뜯어보기에 앞서 스레드라는 것을 알아봅시다

Thread

실행중인 프로그램을 일반적으로 프로세스라 합니다.그리고 우리가 자바프로그램을 가동하면 이 또한 하나의 프로세스가 됩니다. 하지만, 이 자바 프로그램(프로세스)에서 동시에 실행되는 여러 기능을 구현하려면 어떻게 해야할까요? 자바언어에서는 하나의 프로세스 내에서 독립적으로 실행될 수 있는 또 하나의 세부실행단위를 지원하는데 이를 가리켜 쓰레드라 합니다...그리고, OS가 프로세스들을 관리하면 동시에 프로세스를 제어해주듯이,

자바에서는 생성된 쓰레드들을 JVM(일종의 소형 OS)에 맡겨야 동시 실행이 가능하게 됩니다. 즉, 쓰레드를 만들어 시스템에 맡겨야 합니다!

참고로 쓰레드는 쓰레드 클래스를 이용하면 됩니다!

이제 클래스해부를 해봅시다.

상수로 width와 height를 정해주었습니다. 그리고 지금까지 만든 배경화면이랑 주인공이랑 이게임을 자동으로 돌릴 쓰레드를 만들고 적군과 총알을 만들었습니다. 배경이랑 적군은 우리가 개수를 정해서 만들거라 배열로 하였고 총알을 스페이스바를 누를 때마다 발동되도록 설정할거기 때문에 어레이리스트로 만들었습니다!

//생성자

```
	public GamePanel() {
		setPreferredSize(new Dimension(WIDTH, HEIGHT));
		setFocusable(true);
		
		createBg();//배경생성
		createHero();//주인공생성
		createEnemy();//적군생성
		
		gameThread = new Thread() {
			@Override
			public void run() {
				while (true) {
					if (check) {
						gameLoop();
					}
					try {
						Thread.sleep(10);// 게임의 속도 조절 FPS 개념
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		};
		gameThread.start();
		//패널과 리스너와의 연결 : 어댑터라는 리스너..
		this.addKeyListener(new KeyAdapter() {
			public void keyPressed(KeyEvent e) {
				int key=e.getKeyCode();
				
				switch(key) {
					
				case KeyEvent.VK_LEFT :{hero.velX=-5;break;}
				case KeyEvent.VK_RIGHT :{hero.velX=5;break;}
				case KeyEvent.VK_UP:{hero.velY=-5;break;}
				case KeyEvent.VK_DOWN :{hero.velY=5;break;}
				case KeyEvent.VK_SPACE :{createBullet();break;}
				
				
				}
			}
			public void keyReleased(KeyEvent e) {
				int key=e.getKeyCode();
				
				switch(key) {
					
				case KeyEvent.VK_LEFT :{hero.velX=0;break;}
				case KeyEvent.VK_RIGHT :{hero.velX=0;break;}
				case KeyEvent.VK_UP:{hero.velY=0;break;}
				case KeyEvent.VK_DOWN :{hero.velY=0;break;}
				
				
				}
			}
		});
	}
```

게임패널생성자입니다. 사이즈를 정해주고 포커스를 맞춰줍니다. 그리고 생성을 모두 이 클래스의 생성자 안에서 하면 너무 커져버리니까 클래스를 따로 만들어 빼둔 다음 이 생성자에서 호출을 하도록 했습니다! 그리고 게임 쓰레드를 만들었습니다. 물론이 객체는 Thread클래스입니다. Thread는 그 안에 있는 run()메서드를 재정의해서 사용합니다. run()메서드를 보시면 조건부 무한반복으로 gameLoop()메서드를 실행시키는 것을 볼 수있습니다. 이 메서드는 밑에 있지만 안에는 repaint()메서드만 있습니다. 따라서 계속 repaint를 하면서 게임 안에 있는 모든 움직임의 좌표 수정값이 이 repaint에 의해 다시 그려지는 것이죠. 마치 멈추지 않는 심장과도 같습니다. 하지만 멈추는데요 ㅎㅎ 언제냐면 아까 게임메인 메뉴바에서 일시정지가 있었죠? 이 일시정지 아이템에 액션리스너를 달아서 누르면 저 조건부에 있는 check가 false로 되면서 gameLoop()가 실행이 안 됩니다. 그래서 멈추는 것이죠.

Thread.sleep(10)은 무엇일까요? cpu는 우리의 생각 이상으로 빠릅니다. 그래서 속도를 늦춰주지 않으면 게임이 미친듯이 빨리 가는 것이죠. 그래서 저 10자리엔 1000분의 ?초에 해당됩니다. 따라서 1000분의 10초인것이죠 아주 잠깐이지만 이것으로 우리 육안으로 보기 좋게 구동이 될겁니다.

그리고 start()메서드로 이 쓰레드를 jvm에 넘겨줍니다. 그럼 jvm이 알아서 run()메서드를 계속 시키는것이죠.

그리고 그다음엔 패널과 각 리스너와의 연결입니다. 여기선 어댑터를 사용했는데요 어댑터란 만약 키리스너를 implements하면 강제적으로 그 안의 3개의 메서드를 다 구현해야합니다. 그런데 우리는 그중 선택적으로 사용하고 싶을때 그 3개를 다 미리 다 구현한(바디안엔 아무것도 없습니다.) 어댑터를 2차적으로 이용해서 강제성을 없애주는 것입니다. 우리는 keyPressed와 keyreleased로 우주선을 움직이고 그 키를 때면 다시 속도를 0으로 바꿔서 멈추도록 했습니다!

//생성파트

```
	// 배경을 생성
	public void createBg() {
//		Class myClass=this.getClass();

		URL url = this.getClass().getClassLoader().getResource("game_bg.jpg");
		try {// 에러가 날 가능성이 있는 코드..
			BufferedImage buffImg= ImageIO.read(url);
			gameBg[0] = new GameBg(0, 0, 1280, 640, -1, 0, buffImg);
			gameBg[1] = new GameBg(1280, 0, 1281, 640, -1, 0, buffImg);
		} catch (IOException e) {// 혹여나 에러가 나면 비정상적으로 종료하지말고, catch문영역을 수행해..
			e.printStackTrace();
		}


	}
	
	//주인공을 생성
	public void createHero() {
		URL url = this.getClass().getClassLoader().getResource("plane.png");
		try {
			BufferedImage buffImg= ImageIO.read(url);
			hero =new Hero(100,200,120,75,0,0,buffImg);
		} catch (IOException e) {
			e.printStackTrace();
		}

	}
	//적군을 생성
	public void createEnemy() {
		String[] imgName = {"e1.png","e2.png","e3.png","e4.png","e5.png"};
		
		try {
			for (int i = 0; i < imgName.length; i++) {
				BufferedImage buffImg = ImageIO.read(this.getClass().getClassLoader().getResource(imgName[i]));
				enemy[i] = new Enemy(WIDTH,100+(100*i),90,90,-2,0,buffImg);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}

		
	}

	//총알을 생성
	public void createBullet() {
		try {
			BufferedImage buffImg = ImageIO.read(this.getClass().getClassLoader().getResource("ball.png"));
			 bulletArray.add(new Bullet(hero.x+100,hero.y+50,20,20,15,0,buffImg));
		} catch (IOException e) {
			e.printStackTrace();
		}

		
	}
```

생성파트입니다. 여기서 아까 호출한 createBg,Hero등등을 선언했었습니다. 물론 배경과 적군은 배열, 총알은 어레이리스트를 활용했씁니다.

좀 생소한게 있을텐데

```
BufferedImage buffImg = ImageIO.read(this.getClass().getClassLoader().getResource("ball.png"));
```

이 코드입니다. 자세한건 나중에 또 알게되면 올리겠습니다. 일단 버퍼이미지형식의 buffImg를 만들어야했습니다. 왜냐하면 저번처럼 이미지를 가져올때 항상 로컬디렉토리를 사용했는데 이것은 그 이미지 자체를 클래스패스로 이용해서 가져올수 있게 했습니다.

//구동기관

```
	@Override
	//repaint() --> update() --> paint();
	public void paintComponent(Graphics g) {
		g.setColor(Color.YELLOW);
		g.fillRect(0, 0, WIDTH, HEIGHT);
		
		gameBg[0].tick();
		gameBg[0].render(g);
		gameBg[1].tick();
		gameBg[1].render(g);
		
		hero.tick();
		hero.render(g);
		
		//누적된 총알 모두의 tick(),render()호출
		for (int i = 0; i < bulletArray.size(); i++) {
			Bullet bullet =bulletArray.get(i); //요소 꺼내기
			bullet.tick();
			bullet.render(g);
		}
		
		
		for(int i=0;i<enemy.length;i++) {
			enemy[i].tick();
			enemy[i].render(g);
			
		}
	}
```

각 요소를 움직이는 것입니다. 원래는 paint()를 썼었는데 이번엔 업그레이드 된 메서드를 썼습니다!

이제 한번 실행해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/GUI-Game-App-2/1.png)

주인공과 적군이 움직이고 배경도 움직이도 총알도 잘 날아가네요.. 오늘 한거는 반드시 보고 또 봐서 저 혼자 충분히 만들 수 있을 정도로 제것으로 만들어야합니다!