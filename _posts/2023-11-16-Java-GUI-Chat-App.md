---
title: Java GUI로 채팅 어플 구현
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 gui를 가장한 클래스와 객체, 주소값에 대해서 더 깊이 배워보는 시간을 가지겠습니다 ㅋㅋ

패키지안에 2개의 클래스를 만들고 각각의 텍스트 area를 공유하고 채팅시스템을 넣어보겠습니다.

ChatA

```
package com.koreait.project0825.event4;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.FlowLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class ChatA extends JFrame implements ActionListener,KeyListener{
		JTextArea area;
		JScrollPane scroll;
		JTextField t_input;
		JButton bt;
		ChatB chatB; //has a
		
		//생성
		public ChatA() {
			area =new JTextArea();
			scroll = new JScrollPane(area);
			t_input = new JTextField(20);
			bt = new JButton("open");
			
			//스타일 
			area.setPreferredSize(new Dimension(280,320));
			area.setBackground(Color.YELLOW);
			
			//조립
			setLayout(new FlowLayout());//플로우 레이아웃 적용
			add(scroll);
			add(t_input);
			add(bt);
			
			
			//윈도우 설정
			this.setBounds(200,100,320,420);
			setVisible(true);
			setDefaultCloseOperation(EXIT_ON_CLOSE);
			
			
			//리스너와의 연결
			bt.addActionListener(this);
			t_input.addKeyListener(this);
		
		}
	
	public static void main(String[] args) {
		new ChatA();
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		//대화 상대방 띄우기
		if(chatB == null) {
			chatB = new ChatB();
			chatB.setChatA(this);
		}
	}

	@Override
	public void keyPressed(KeyEvent e) {;}
	@Override
	public void keyReleased(KeyEvent e) {
		if(e.getKeyCode()==KeyEvent.VK_ENTER) {
			String msg = t_input.getText();//입력값 가져오기
			chatB.area.append(msg+"\n");//ChatB area에 출력
			this.area.append(msg+"\n");//나의 area에 출력
			
			t_input.setText("");//입력했던 데이터 다시 지우기
		}
	}
	@Override
	public void keyTyped(KeyEvent e) {;}
}
```

ChatA클래스입니다. 임포트문에 엄청 많은 것들이 보이는데요, 원래 저번 시간에 gui에선 java.awt 기능들을 많이 사용했는데 이번에는 조금 업그레이드된 버전인 javax.swing패키지에 있는 기능들을 사용하겠습니다. 그래서 상속도 JFrame으로 하는 모습, 그리고 ActionListener,KeyListener 인터페이스를 이용하네요, 리스너에 관해선 후에 설명하겠습니다. 일단 먼저 채팅 메신저를 만들건데 버튼과 텍스트를 칠 텍스트필드, 그리고 채팅기록을 넣을 textarea, 그리고 스크롤 기능도 있어야겠지요?

```
        JTextArea area;
		JScrollPane scroll;
		JTextField t_input;
		JButton bt;
```

각 클래스의 객체를 만들었습니다. 생성자 호출은 어디서 할까요?이 클래스 생성자에 넣을 예정입니다. 왜냐면 메인메서드 부분에서 다 객체를 적을 필요없이 생성자 호출로서 실행하도록 만들것이기 때문입니다.

```
			area =new JTextArea();
			scroll = new JScrollPane(area);
			t_input = new JTextField(20);
			bt = new JButton("open");
```

그래서 생성자 ChatA() 바디 안에 다 넣었습니다. 스크롤은 독특하게 textarea를 품습니다. 하나의 세트가 된 셈이죠. 그리고 텍스트필드의 사이즈는 20으로 했습니다. 버튼에는 "open"이라는 문자열을 넣었습니다. 앞으로 이 버튼을 누르면 다른 클래스의 메신저 창이 나와서 서로 소통하게 말이죠.

저는 생성자에서 작업순서를 스타일(각 컴포넌트 설정) - 조립(컴포넌트와 프레임)- 윈도우 설정(frame 설정) -연결 (후에 설명할 리스너와의 연결) 순서대로 하겠습니다.

```
			//스타일 
			area.setPreferredSize(new Dimension(280,320));
			area.setBackground(Color.YELLOW);
			
			//조립
			setLayout(new FlowLayout());//플로우 레이아웃 적용
			add(scroll);
			add(t_input);
			add(bt);
```

스타일과 조립파트입니다. 텍스트 area의 사이즈를 설정하는 setPreferredSize()메서드입니다. 매개변수엔 Dimension타입을 넣는 건데 또 밖에서 객체 만들고 넣기보단 그냥 바로 new연산자로 넣었습니다.그리고 Dimension()생성자 매개변수엔 280,320인데요 쉽게 말해서 가로 세로 크기입니다. 그리고 setBackground()메서드로 Color타입의 상수 YELLOW를 넣어서 앞으로 우리가 채팅할 메세지들이 보이는 영역을 노란색으로 했습니다.

그리고 조립파트입니다. add메서드가 보이는데 선언하지도 않았는데 어떻게 그냥 쓸까요? 답은 상속입니다. JFrame을 상속받았으니 JFrame의 메서드들을 자기가 쓸 수 있죠. 그래서 this. 를 쓰는데 이건 생략해도 되는 것이기 때문에 그냥 setLayout과 add만 달랑써도 사용이 가능합니다.

레이아웃을 flowlayout으로 바꾸고 area를 품고있는 scroll과 텍스트 필드와 버튼을 JFrame에 붙였습니다.

```
            //윈도우 설정
			this.setBounds(200,100,320,420);
			setVisible(true);
			setDefaultCloseOperation(EXIT_ON_CLOSE);
			
			
			//리스너와의 연결
			bt.addActionListener(this);
			t_input.addKeyListener(this);
```

그 다음엔 윈도우 설정과 리스너 연결을 해보겠습니다. JFrame을 설정해봅시다. setBounds(200,100,320,420) 에서 인수를 하나씩봅시다.

앞에 두 숫자는 시작좌표입니다. 시작점은 프로그램 상단 좌측을 시작점이며 이 시작점 포인트를 가리키는 것이죠. 따라서 200,100은 컴퓨터 화면의 시작점인 화면 좌측 최상단에서부터 가로,세로 200,100씩 떨어진 곳에 프로그램 시작점으로 설정하는 것입니다. 그 뒤에 320,420은 그 프로그램 사이즈입니다. 그리고setVisible(true)로 보이게 하고 setDefaultCloseOperation(EXIT\_ON\_CLOSE)은 무엇일까요? 프로그램 실행하면 콘솔창과 별도로 gui프로그램이 따로나옵니다. 그런데 이 창을 x를 눌러서 끌 수 가 없게되있습니다. 그것을 x를 눌렀을 때 콘솔창을 포함한 전체 프로그램을 종료시키는 것입니다.

이제 리스너와 연결파트입니다. 리스너란 무엇일까요? 우리가 키보드나 마우스 클릭 등 이런 것들은 전부 os에서 jvm으로 그 신호를 보냅니다. 그런데 리스너를 설정하지 않은 디폴트값은 이러한 하나하나의 신호들을 전부 우리에게 보여주지 않죠. 따라서 처음 부분에 이 클래스가 상속받은 것도 있지만 인터페이스를 이용하는 것도 있었죠? 따라서 리스너는 인터페이스 형태로 이용을 한다면 그 인터페이스 안에 있는 추상 메서드들을 구현해야겠죠? 이클립스엔 클릭 몇번으로 안에 있는 것들을 꺼내줍니다.

```
	@Override
	public void actionPerformed(ActionEvent e) {
		//대화 상대방 띄우기
		if(chatB == null) {
			chatB = new ChatB();
			chatB.setChatA(this);
		}
	}

	@Override
	public void keyPressed(KeyEvent e) {;}
	@Override
	public void keyReleased(KeyEvent e) {
		if(e.getKeyCode()==KeyEvent.VK_ENTER) {
			String msg = t_input.getText();//입력값 가져오기
			chatB.area.append(msg+"\n");//ChatB area에 출력
			this.area.append(msg+"\n");//나의 area에 출력
			
			t_input.setText("");//입력했던 데이터 다시 지우기
		}
	}
	@Override
	public void keyTyped(KeyEvent e) {;}
```

이부분입니다. @Override로 표현해주고있습니다.

먼저 클릭할 때 신호를 감지하는 리스너는 무엇일까요? actionPerformed()메서드입니다. 이것을 포함하는 인터페이스도 이거 하나만 갖고있습니다.

여기엔 어떤 기능일까요?

```
	@Override
	public void actionPerformed(ActionEvent e) {
		//대화 상대방 띄우기
		if(chatB == null) {
			chatB = new ChatB();
			chatB.setChatA(this);
		}
	}
```

액션이벤트가 발생할때 조건문을 거쳐 앞으로 만들 ChatB 클래스의 객체 chatB가 널값이라면 안에 생성자를 호출합니다. 이말은? ChatB클래스도 이 클래스와 비슷하게 만들어서 생성자가 호출되면 모든 프로그램이 실행되도록 설계할 것이죠. 그리고 chatB.setChatA(this)은 무엇일까요?ChatB클래스에 ChatA(이 클래스) 를 new없이 그대로 가져와야하기 때문입니다. 그러면 ChatB클래스에 ChatA주소값은 이 클래스의 그 주소값을 가져가겠죠. 이것을 나중에 버튼과 연결시켜 버튼을 누르면 ChatB생성자를 호출하고 자신의 주소값을 또한 ChatB에 넘겨주겠죠? 재밌습니다 ㅎㅎ

```
	@Override
	public void keyPressed(KeyEvent e) {;}
	@Override
	public void keyReleased(KeyEvent e) {
		if(e.getKeyCode()==KeyEvent.VK_ENTER) {
			String msg = t_input.getText();//입력값 가져오기
			chatB.area.append(msg+"\n");//ChatB area에 출력
			this.area.append(msg+"\n");//나의 area에 출력
			
			t_input.setText("");//입력했던 데이터 다시 지우기
		}
	}
	@Override
	public void keyTyped(KeyEvent e) {;}
```

이것은 key리스너입니다. 우리가 키보드를 사용할 때 신호를 감지하는 리스너입니다. 이 인터페이스에는 3개의 추상메서드가 있습니다. 바로

keyPressed,keyReleased,keyTyped 입니다. 각각 키보드의 어떤 버튼을 누를 때, 누르고 땔 때, 그리고 어떤 것이 입력될 때(누르지 않아도) 발동하는 메서드들입니다. 그리고 매개변수 keyevent e 에는 우리가 입력할 때의 모든 정보를 가지고 있습니다. 저 e를 그냥 출력문에 넣고 출력하면 엄청나게 긴 정보가 출력됩니다.. 그런데 우리는 혼란을 피하기 위해 keyReleased()메서드만 사용하겠습니다. keyevent타입 안에 있는 메서드엔 getKeyCode()가 있습니다. 이것은 우리가 입력한 문자를 아스키코드로 정수로 변환해서 알려주는 메서드입니다. 이것을 왜 넣었냐면 우리가 어떤 문장을 쓰고 엔터를 칠때 이것을 텍스트area로 전달하기 위함입니다. 저게 없으면 한 문자씩 칠때마다 그것이 올라가겠죠? 그리고 출력문으로 실험을 해보면 엔터키는 정수 10으로 표현됩니다. 그리고 java는 이것을 헷갈리지 않게 상수 VK\_ENTER로 정했습니다. 그래서 엔터를 칠때 저 if문 바디가 실행되겠습니다. 문자열 msg를 선언하고 여기엔 area타입의 객체 getText()를 사용했습니다. 이것은 우리가 입력했던 값들을 저장하고 있는 메서드입니다. 따라서 우리가 엔터를 치면 그동안 입력했던 문자열들이 msg로 들어오죠. 그리고 이것을 이 클래스의 메신저 창과 ChatB메신저에도 보이게 해야하니 chatB.area.append(msg+"\\n")로 한줄씩 출력되도록 하였습니다. append가 아니라 setText()면 덮어쓰기가 되어서 전에 썼던게 없어지고 새로운것만 보이게 됩니다. 따라서 append()를 썼습니다. this로 자기자신에게도 마찬가지로 출력되도록 하였습니다.

```
			//리스너와의 연결
			bt.addActionListener(this);
			t_input.addKeyListener(this);
```

이제 힘들게 만든 리스너들을 각각 버튼과 텍스트 필드에 연결시키겠습니다. 메서드는 add+리스너()입니다. 그리고 매개변수엔 자기를 넣었습니다!

이제 ChatB도 볼까요? 비슷해서 몇개만 간단히 볼게요

```
package com.koreait.project0825.event4;

import java.awt.Color;
import java.awt.Dimension;
import java.awt.FlowLayout;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class ChatB extends JFrame implements KeyListener{
		JTextArea area;
		JScrollPane scroll;
		JTextField t_input;
		ChatA chatA;
		
		
		//생성
		public ChatB() {
			area =new JTextArea();
			scroll = new JScrollPane(area);
			t_input = new JTextField(20);
			
			//스타일 
			area.setPreferredSize(new Dimension(280,320));
			area.setBackground(Color.cyan);
			
			//조립
			setLayout(new FlowLayout());//플로우 레이아웃 적용
			add(scroll);
			add(t_input);
			
			//윈도우 설정
			this.setBounds(550,100,320,420);
			setVisible(true);
			
			
			//리스너와 연결
			t_input.addKeyListener(this);
		
		
		
		
		}
		//누군가에 의해 ChatA를 넘겨받을 수 있는 setter메서드 정의
		public void setChatA(ChatA chatA) {
			this.chatA = chatA;
		}
		
		@Override
		public void keyPressed(KeyEvent e) {;}
		@Override
		public void keyReleased(KeyEvent e) {
			if(e.getKeyCode()==KeyEvent.VK_ENTER) {
				String msg = t_input.getText();//입력값 가져오기
				chatA.area.append(msg+"\n");//ChatA area에 출력
				this.area.append(msg+"\n");//나의 area에 출력
				
				t_input.setText("");//입력했던 데이터 다시 지우기
			}
		}
		@Override
		public void keyTyped(KeyEvent e) {;}
}
```

여기서

```
		//누군가에 의해 ChatA를 넘겨받을 수 있는 setter메서드 정의
		public void setChatA(ChatA chatA) {
			this.chatA = chatA;
		}
```

이부분만 보겠습니다. 아까 ChatA에서 버튼을 누르면 이 ChatB의 생성자를 호출함과 동시에 이 메서드에 매개변수로 ChatA의 주소값이 오게됩니다. 그리고 그 주소값을 여기서 사용합니다. 따라서 서로 주소값을 서로 갖고있는것이죠. ChatA에 메인메서드가 있으니 거기서 실행하겠습니다.!

![Desktop View](/assets/img/Programming-Language/Java/GUI-Chat-App/1.png)

왼쪽 노란색이 ChatA의 메신저창입니다. 저 버튼 open을 누르면 오른쪽 하늘색의 ChatB메신저창이 나옵니다. 뭐라도 입력할까요?

![Desktop View](/assets/img/Programming-Language/Java/GUI-Chat-App/2.png)

ChatA에서 안녕? 이라는 문자를 입력하고 엔터를 눌러보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/GUI-Chat-App/3.png)

보시면 두 메시전의 텍스트 area에 정상 출력되는것이 보입니다 ㅎㅎ 재밌네요

이번시간에는 여기까지 하겠습니다!