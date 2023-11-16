---
title: Java GUI로 게임 만들기 1
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
컬렉션 프레임워크란? 객체를 모아서 처리하는데 유용한 객체들을 지원해주는 라이브러리입니다. 이번 시간에는 이 프레임웍중 많이 쓰이는 3개를 알아보겠습니다.

**Set**

순서가 없는 객체를 처리하는 인터페이스입니다. 인덱스(순서)가 없어서 혼자서 추출은 못하지만 어떤 객체가 있는지 없는지에 대해선 엄청 빨리 찾습니다. 일단 인터페이스 이므로 객체를 따로 만들지는 않습니다. 대신 그 밑에 HashSet을 이용해서 사용합니다. 바로 보시죠

```
package com.koreait.project0826.collection;

import java.util.HashSet;
import java.util.Iterator;
import java.util.Set;

//객체를 모아서 처리하는데 유용한 객체들을 지원해주는 라이브러리인 컬렉션 프레임웍
//중 순서가 없는 객체를 처리하는 Set을 배워보자!
public class SetStudy {

	public static void main(String[] args) {
		Set set= new HashSet();
		String s = "사과";
		
		
		set.add(s);//Object형을 넣을 수 있다는 것은 넣을 엘리먼트에 제한이 없다..
		set.add("오렌지");
		set.add("딸기");
		set.add("포도");
		set.add("수박");
        Iterator it = set.iterator();
		
		while(it.hasNext()) {
			String e =(String)it.next();//요소를 꺼낸다
			System.out.println(e);
		}
		
```

순서없이 모여있는 객체를 모두 출력해보려고하는데요, 총 몇개이고, 반복문으로 가져오는 방법은 사실 Set은 순서가 없으므로, 반복문등을 이용하여 처리하려면 순서 있게 정렬해야 합니다 이 때 사용하는 객체가 바로 Iterator입니다. Iterator(jdk5)이전에는 Enumeration이 사용됨

그래서 set 안에 있는 iterator()메서드를 사용해서 Iterator 타입의 객체 it에 그 순서를 넣어줍니다. 그리고 iterator안에는 hasnext()와 next()가 있습니다. 전자는 논리형으로 그 다음 값이 있으면 true를 반환합니다. next()는 다음 값을 가져옵니다. 그래서 얼마나 반복할지 모르므로 while문을 사용해서 set에 있는 데이터들을 출력합니다.

**List**

```
package com.koreait.project0826.collection;

import java.util.ArrayList;
import java.util.List;

import javax.swing.JButton;

public class ListStudy {
	
	public static void main(String[] args) {
		
		List<String> list = new ArrayList<String>(); //크기를 명시하지 않아도 됨.. 즉 크기가 유동적으로 늘어남
		//Generic Type : 컬렉션 프레임웍의 요소를 제한을 가하는 방법
		
		
		list.add("바나나");
		list.add("멜론");
		list.add("파인애플");
		list.add("자두");
		list.add("대추");
		list.add("스뚜링");
		
		for (int i = 0; i < list.size(); i++) {
			System.out.println(list.get(i));
		}
		
		//컬렉션 프레임웍 사용시 보다 개선된 improve for문이 지원됨(jdk 5버전부터)
		for(String obj : list) {
			System.out.println(obj);
		}
	}
}
```

컬렉션 프레임웍이 지원하는 객체 중 List계열의 객체를 배워보도록 하겠습니다. 참고로 List는 순서가 있는 객체집합을 처리하므로, 배열과 거의 같습니다. 그런데 이걸 왜 사용하냐고요?배열은 생성시 크기를 지정해야합니다. 하지만 list는 크기를 지정하지 않아도 유동적으로 값을 더 할 때마다 그 공간을 늘려줍니다. 그리고 배열은 기본자료형 및 객체자료형을 대상으로 데이터를 넣지만, 컬렉션 프레임웍의 대상은 오직 객체만을 대상으로 합니다.

그 중 가장 많이 사용되는 ArrayList를 사용하겠습니다. 먼저

List<String> list = new ArrayList<String>(); 로 객체를 만들어 줍니다. 여기서 꺽새 <>가 보이는데 이것은 제네릭형을 넣을 때 씁니다. 따라서 어떤 타입과 같이 묶어서 사용됩니다. 즉 이 list객체를 문자열만 입력받도록 강제하는 장치입니다. 그리고 arrylist의 메서드 add()로 하나씩 추가합니다. 이때 중요한건 추가를 할 때마다 그 공간을 알아서 만들어줍니다. 기특하죠?안에 있는 데이터를 가져올 땐 get()메서드를 통해서 가져옵니다.

**Map**

```
package com.koreait.project0826.collection;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Set;


//
public class MapStudy {
	public static void main(String[] args) {
		Map<String,String> map = new HashMap<String,String>();
		
		map.put("name1","Smith");
		map.put("name2","Scott");
		map.put("name3","King");
		
		System.out.println(map.get("name1"));
		Set<String> keySet = map.keySet();//명단만 끄집어 내서 Set에 담는다.
		Iterator<String> it = keySet.iterator();//명단을 일렬로 늘어뜨림..
		
		
		while(it.hasNext()) {
			System.out.println(map.get(it.next()));
		}
		
	}
}
```

컬렉션 프레임웍이 지원하는 객체 중 key-value의 쌍으로 구성된 집합을 제어하는 Map계열입니다. 마치 열쇠와 자물쇠로 관계가 정립된 커플끼리 데이터가 저장됩니다. 따라서 key값만 가져와도 value를 알 수 있습니다. 이 Map 중에서도 가장 많이 쓰이는게 HashMap입니다. 역시 쌍으로 구성되어 있어야 하기 때문에 값을 입력할 때도

```
Map<String,String> map = new HashMap<String,String>();
```

이와 같이 key와 value를 둘다 문자열로 넣는 것이죠. 그리고

```
System.out.println(map.get("name1"));
```

이것으로 출력을 합니다.

이제 java에서 지원하는 graphics툴로 우리만의 게임을 한번 만들어 봅시다!

현실에서의 그래픽의 구성요소와 프로그래밍 언어에서의 그래픽의 구성요소간 이해를 간단히 정리하겠습니다.

현실 전산

그래픽의 주체 : 화가 ---- UI컴포넌트

그래픽 그림의 대상 : 도화지--- UI컴포넌트

팔레트 : 팔레트(색)--색상,도형,문자열,이미지.. Graphics 객체로 지원

그리는 행위: 붓 --------paint()메서드

지금까지는 이미 그려진 것들을 우리가 사용만 했었다면 우리가 이제 컴포넌트가 그리는 것에 개입을 해봅시다. 전산이 그릴 때 호출하는 메서드가 paint()입니다. 우리는 이것을 재정의 하면서 그리는 데에 방해(?)를 하는 것입니다. paint메서드는 그림그리는 행위에 불과하고, 사실상의 그림의 효과는 팔레트인 Graphics객체가 전담합니다.

이미지를 메모리에 올릴 때는 두 개의 객체를 사용합니다. Image 타입과 Toolkit이 그것이죠. 먼저 Tookit은 로컬저장소에 있는 파일을 디렉토리의 형태로 불러올 수 있습니다. 그리고 그것을 Image타입의 객체에 전달하면 우리가 이미지를 가져올 수 있습니다.

자세한 설명은 코드를 보면서 설명 드릴게요~ 간단한 게임을 만들어볼건데 4개의 클래스를 일단 만들었습니다.

//AnimationApp

```
package com.koreait.project0826.graphic4;

import javax.swing.JFrame;

public class AnimationApp extends JFrame{
	GamePanel gamepanel;
	
	public AnimationApp() {
		gamepanel = new GamePanel();
		add(gamepanel);//center
		
		
		this.pack(); //프레임 안쪽의 내용물 만큼 줄어든다..
		//프레임을 모니터의 정중앙에 오게하는 법
		this.setLocationRelativeTo(null);
		setVisible(true);
		setDefaultCloseOperation(EXIT_ON_CLOSE);
		
	}
	public static void main(String[] args) {
		new AnimationApp();
	}
}
```

첫번째 클래스는 실행부가 담겨있는 AnimationApp클래스입니다. JFrame을 상속받고 앞으로 살펴볼 GamePanel 를 만들고 객체 생성을 이 클래스의 생성자 부분에서 해줍니다. 그리고 이 프레임에 gamepanel을 부착했습니다. 그리고 this.pack()이 보이는데요 이것은 그 패널의 크기만큼 프레임의 크기를 맞춰주는 메서드입니다. 그리고 this.setLocationRelativeTo(null);로 창이 켜지면 화면 중간에 켜지도록 하였습니다.

//GamePanel

```
package com.koreait.project0826.graphic4;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import java.awt.Image;
import java.awt.Toolkit;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

import javax.swing.JPanel;

public class GamePanel extends JPanel implements KeyListener{
	public static final int WIDTH=1280;
	public static final int HEIGHT=640;
	Toolkit kit = Toolkit.getDefaultToolkit();
	Image bg_img;//게임 배경 이미지
	Image hero_img;
	Hero hero;
	Image[] enemy_image=new Image[5];
	Enemy[] enemy = new Enemy[5];
	
	public GamePanel() {
		this.setPreferredSize(new Dimension(WIDTH,HEIGHT));//너비와 높이 지정
		bg_img= kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\bg.jpg");
		hero_img= kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\plane.png");
		enemy_image[0]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e1.png");
		enemy_image[1]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e2.png");
		enemy_image[2]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e3.png");
		enemy_image[3]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e4.png");
		enemy_image[4]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e5.png");
		
		//주인공 생성
		hero= new Hero(hero_img,100,150);
		
		//적군 생성
		enemy[0]= new Enemy(enemy_image[0],WIDTH-100,100);
		enemy[1]= new Enemy(enemy_image[1],WIDTH-100,200);
		enemy[2]= new Enemy(enemy_image[2],WIDTH-100,300);
		enemy[3]= new Enemy(enemy_image[3],WIDTH-100,400);
		enemy[4]= new Enemy(enemy_image[4],WIDTH-100,500);
		
		//리스너와 연결
		addKeyListener(this);
		
		//프로그램 가동과 동시에 포커스가 프레임에 활성화되어 있기에, 현재 패널로 포커스를 이동시키자!
		this.setFocusable(true);
	
	}
	
	@Override
	public void paint(Graphics g) {
		g.drawImage(bg_img, 0, 0, WIDTH, HEIGHT, this);
		hero.render(g);
		for (int i = 0; i < enemy.length; i++) {
			enemy[i].render(g);
		}
	}

	@Override
	public void keyPressed(KeyEvent e) {
		if(e.getKeyCode()==e.VK_RIGHT) {
			hero.x +=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_LEFT) {
			hero.x -=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_UP) {
			hero.y -=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_DOWN) {
			hero.y +=10;//10누적
		}
		repaint();
	}

	@Override
	public void keyReleased(KeyEvent e) {;}

	@Override
	public void keyTyped(KeyEvent e) {;}
	
	
}
```

좀 길죠? 왜냐하면 이 패널에 이 게임의 부속품들이 들어올 것이기 때문입니다. 패널의 크기를 고정 시키도록 하기 위해서 상수로 WIDTH와 HEIGHT를 지정했습니다. 그리고 아까 말씀드린 Toolkit과 Image 파일 객체를 만듭니다. Toolkit은 인터페이스므로 자체 객체를 new연산자로 만드는게 아닌 이미 있는 Toolkit.getDefaultToolkit()를 통해서 인스턴스를 가져옵니다. 그리고 bg\_img는 게임 배경이미지를 할것이고 hero\_img는 이 게임의 주인공입니다. 그리고 hero랑 enemy는 여기서 만들기보단 따로 클래스를 만들어서 독립적인 개성을 갖도록 하겠습니다. 특징은 적들은 한명이 아니라 여러 명일 예정이라 배열로 만들었습니다.

//GamePanel 생성자

```
	public GamePanel() {
		this.setPreferredSize(new Dimension(WIDTH,HEIGHT));//너비와 높이 지정
		bg_img= kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\bg.jpg");
		hero_img= kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\plane.png");
		enemy_image[0]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e1.png");
		enemy_image[1]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e2.png");
		enemy_image[2]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e3.png");
		enemy_image[3]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e4.png");
		enemy_image[4]=kit.getImage("D:\\bigData\\workspaceOfJava2\\project0826\\img\\shooting\\e5.png");
		
		//주인공 생성
		hero= new Hero(hero_img,100,150);
		
		//적군 생성
		enemy[0]= new Enemy(enemy_image[0],WIDTH-100,100);
		enemy[1]= new Enemy(enemy_image[1],WIDTH-100,200);
		enemy[2]= new Enemy(enemy_image[2],WIDTH-100,300);
		enemy[3]= new Enemy(enemy_image[3],WIDTH-100,400);
		enemy[4]= new Enemy(enemy_image[4],WIDTH-100,500);
		
		//리스너와 연결
		addKeyListener(this);
		
		//프로그램 가동과 동시에 포커스가 프레임에 활성화되어 있기에, 현재 패널로 포커스를 이동시키자!
		this.setFocusable(true);
	
	}
```

생성자 부분을 보겠습니다. 먼저 setPreferredSize()메서드로 크기를 아까 지정한 상수로 정합니다. 그리고 이미지를 하나씩 메모리에 올린건데요. 먼저 이미지들은 stack처럼 나중에 온것이 기존 것들을 덮어 씌우는 형식이라 배경화면인 bg\_img에 툴킷타입으로 만든 객체 kit으로 getImage()메서드를 통해 디렉토리를 설정하고 그 이미지를 가져와 image타입의 객체들에게 넣어줄 것입니다. 그리고 배열로 선언한 적군 5기의 이미지도 가져옵니다. 이미지를 가져왔으면 주인공과 적군을 생성해야겠지요? 각각의 클래스는 밑에서 설명하겠습니다.

hero= new Hero(hero\_img,100,150);은 hero클래스의 생성자에 있는 매개변수들에게 값을 줍니다. 미리 메모리에 올린 이미지를 넣어주고 시작 좌표를 설정했습니다. 마찬가지로 적군 배열의 각 데이터들한테도 new연산자로 객체를 생성합니다. 적군의 생성자의 매개변수의 형식은 동일합니다. 우리는 이 주인공을 키보드의 방향키로 움직일것입니다. 그래서 미리 선언한 키리스너를 연결시킵니다. 그리고 this.setFocusable(true);가 보이는데요, 이것은 처음 프로그램을 실행시키면 활성화된 것은 프레임, 즉 윈도우창입니다. 이것을 패널에 활성화를 주기 위해서 적었습니다.

//리스너들과 paint()재정의

```
@Override
	public void paint(Graphics g) {
		g.drawImage(bg_img, 0, 0, WIDTH, HEIGHT, this);
		hero.render(g);
		for (int i = 0; i < enemy.length; i++) {
			enemy[i].render(g);
		}
	}

	@Override
	public void keyPressed(KeyEvent e) {
		if(e.getKeyCode()==e.VK_RIGHT) {
			hero.x +=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_LEFT) {
			hero.x -=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_UP) {
			hero.y -=10;//10누적
		}
		else if(e.getKeyCode()==e.VK_DOWN) {
			hero.y +=10;//10누적
		}
		repaint();
	}

	@Override
	public void keyReleased(KeyEvent e) {;}

	@Override
	public void keyTyped(KeyEvent e) {;}
```

이제 paint()메서드를 재정의해서 툴킷으로 이미지 객체에 가져온 이미지 파일들을 본격적으로 패널에 올려봅시다. 이 메서드는 단지 그리는 행위만이고 실질적인 재료를 설정하고 어떻게 할지는 Graphics가 관장합니다. 그래서 그 객체 g로 drawImage()메서드를 사용해서 배경 이미지를 넣어줍니다. 이 메서드의 매개변수를 보면 먼저 사용할 이미지 파일을 정하고 좌표를 정합니다 그리고 사이즈를 정한뒤 마지막에 this가 보이는군요.. 일단 자기자신이란건 알겠는데 여기엔 어떤 것이 왜 와야할까요? 이 자리는 이 메서드를 감시할 주체를 정하는 것입니다. 자세한건 추후에 설명하겠습니다. 지금은 그냥 감시자를 자기자신으로 설정했다는 것만 알아두면 좋을 것같습니다. 그리고 hero.render(g);가 보이네요.. 아마 hero클래스에 있는 히어로 이미지를 올려주는 메서드인 것같네요. 마찬가지로 적군들도 for문을 이용해서render()메서드를 사용하는 것이 보입니다.

마지막으로 이 주인공을 움직일 액션이벤트와 리스너를 소개하겠습니다. 먼저 방향키로 주인공을 움직일 것이므로 getKeyCode()메서드를 방향키의 코드와 비교하여 움직이겠습니다. 오른쪽 방향키를 눌렀을 때 hero.x +=10;이 실행되도록 했습니다. 이것은 hero클래스에 있는 hero객체의 x좌표를 크게하여 오른쪽으로 살짝 움직이게 했습니다. 나머지 방향키로 같은 원리로 작성하였습니다.

//Hero 클래스

```
package com.koreait.project0826.graphic4;

import java.awt.Graphics;
import java.awt.Image;

//객체로 존재해야, 제어할 수 있으므로 주인공을 표현한 클래스를 정의한다.
public class Hero {
	Image hero_img;//주인공 우주선 이미지
	int x;
	int y;
	
	public Hero(Image hero_img,int x,int y) {
		this.hero_img= hero_img;
		this.x = x;
		this.y =y;
	
	}
	//물리적 변화를 줄 수 있는 메서드(x,y축의 값을 변경하는 메서드...)
	public void tick() {
		
	}
	
	public void render(Graphics g) {//물리적 변화를 준, 대상객체를 화면에 그리는 메서드
		//우주선 올려놓기!!
		g.drawImage(hero_img, x, y, 120, 75, null);
	}
}
```

이제 Hero클래스입니다. 보시면 Image타입의 객체가 보이는군요 그리고 주인공의 움직임을 맡아줄 좌표 x,y가 보입니다. 그리고 주인공의 생성자엔 매개변수를 주었습니다. 아까 보았죠? Image타입, 그리고 x,y로 매개변수를 설정하여 이 클래스를 호출할 때 이미지와 x,y좌표를 태어날 때부터 가져오는 것이죠. 그리고 아까 보기만 했던 render()메서드입니다. 오호 drawImage()메서드가 여기 숨어있었군요.. 그리고 아까 생성자로부터 받아온 이미지 파일과 좌표를 여기에 넣는군요.. 이제 이 메서드로 인해서 패널에 이 주인공을 올릴 수 있네요.!

```
package com.koreait.project0826.graphic4;

import java.awt.Graphics;
import java.awt.Image;

//적군을 정의한다!!
public class Enemy {
	Image enemy_img;//주인공 우주선 이미지
	int x;
	int y;
	
	public Enemy(Image enemy_img,int x,int y) {
		this.enemy_img= enemy_img;
		this.x = x;
		this.y =y;	
	
	}
	//적군의 물리량 변화를 줄 메서드
	public void tick() {
		
	}
	
	//화면에 그려질 메서드
	public void render(Graphics g) {
		g.drawImage(enemy_img,x, y, 120, 75, null);
	}
}
```

적군 클래스입니다. 이것은 주인공 클래스 복붙에서 살짝 바꾼겁니다. 다른 점만 확인하죠.. 없습니다 ㅋㅋㅋ 이제 실행시켜볼까요?

![Desktop View](/assets/img/Programming-Language/Java/GUI-Game-App/1.png)

ㅋㅋㅋㅋ 주인공 1기와 적군 5기가 잘 나오네요 ㅎㅎ 심지어 방향키로 움직일 수도 있습니다!!

이번엔 여기까지 하도록 하겠습니다.