---
title: 추상클래스 인터페이스 JAVA GUI
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 추상클래스와 인터페이스, 그리고 GUI 를 좀 만져보겠습니다.

추상클래스는 예전에도 포스팅한적이 있었죠? 추상메서드가 있는 클래스를 추상클래스라고 합니다. 추상메서드란 바디({})가 없는 클래스입니다. 따라서 추상 클래스에게 상속을 받는 클래스들은 구체적인 내용이 없는 클래스를 강제적으로 내용을 구현시켜야 합니다. 일종의 틀이라고 할 수 있죠. 인터페이스는 이런 기능만 전담하는 틀입니다. 이거 끝나고 설명 드릴게요.

```
package com.koreait.music;
/* 이 클래스는 모든 사원이 구현할 음향기기 클래스의 들어갈 필수적인
기능을 강제할 수 있는 역할을 수행할 클래스!
*/

//이 클래스는 의도하여 불완전하게 정의한 것이다!!
public abstract class MusicPlayer{
	// mp3 지원하는 기능
	// 볼륨을 올리는 기능
	// 볼륨을 내리는 기능

	//abstract 수식자를 이용하면, 컴파일러가 이 메서드를 추장메서드로 인식..
	public abstract void supportMp3();//미래의 어느 시점에 각각의 개발자들이 어떤 기기를
	//만들지 예측할 수 없으므로, 코드 내용 또한 해당 기기마다 적절한 예측불가...
	public abstract void volumeUP();
	public abstract void volumeDown();

}
```

MusicPlayer 라는 추상클래스를 만들었습니다. 추상메서드를 포함하고 있기 때문에 class 앞에 abstract를 붙였습니다. 그리고 내용을 쭉 보시면 모든 메서드가 이름선언만 되어있고 구체적인 내용은 담고 있지 않습니다. 따라서 이 클래스를 상속받으면 저 3개의 메서드는 강제적으로 다시 재정의 해야겠죠?바로 오버라이딩입니다.

우리가 만들고 싶은건 블루투스 스피커입니다. 스피커니까 당연히 MusicPlayer안에 속하는 개념이겠죠? 그런데 우리는 이 블루투스 스피커는 날기도하고 킥보드처럼(?)탈 수도 있게... 만들고 싶습니다 ㅋㅋ 좀 이상하지만 한번씩 보죠

```
package com.koreait.fly;


//인터페이스는 오직 추상 메서드만을 보유해야 하므로, 개발자가 굳이 
//메서드에 abstract 수식자를 명시할 필요가 없다...
public interface Drone{

	public void fly();
}
```

다른 fly패키지에 Drone이라는 인터페이스를 만들었습니다. 여기서 잠깐, 인터페이스란? 추상클래스는 그 클래스 안에 추상메서드가 있으면 추상클래스라고 하지만 인터페이스는 아예 추상메서드만을 넣을 수 있습니다. 말 그대로 '틀'의 역할만 하는 것이죠. 왜 Drone을 추상클래스로 안 만들고 인터페이스로 만들었는지는 후에 알려드리겠습니다.

```
package com.koreait.ride;


//타고다닐 수 있는 객체를 정의
public interface Roller {

	public abstract void run();//추상메서드 정의
}
```

그리고 또 다른 패키지 ride에 Roller라는 인터페이스를 또 만들었습니다. 이제 MusicPlayer를 상속받으면서 날기도하고 타기도하는 요상한 블루투스 스피커를 만들어봅시다!

```
package com.koreait.music;

import com.koreait.fly.Drone;
import com. koreait.ride.Roller;

public class BSpeaker extends MusicPlayer implements Roller,Drone{
					 /*is a*/			/*is a*/
	public void connect(){
		System.out.println("근거리 블루투스 네트웍 연결");
	}
	//오버라이딩..
	public void supportMp3(){
		System.out.println("블루투스를 연동한 MP3파일 로드");
	}

	public void volumeUP(){
		System.out.println("블루투스를 연동한 볼륨 올리기");
	}
	public void volumeDown(){
		System.out.println("블루투스를 연동한 볼륨 내리기");
	}

	public void run(){
		System.out.println("달린다.");
	}

	public void fly(){
		System.out.println("날다");
	}
}
```

자 여기서 아까 2개를 인터페이스로 만든 비밀이 나옵니다.

```
public class BSpeaker extends MusicPlayer implements Roller,Drone
```

자바에선 다중상속을 금지시키고 있습니다. 하지만 현실적으로 다중상속을 하지 않고는 현대 시대의 현실을 반영하기가 힘듭니다. 그래서 타협을 한거죠. 인터페이스를 만들면 상속이 아니라 저렇게 implements로 이용합니다. 그리고 인터페이스는 다중이용이 가능하죠. 그래서 이 블루투스 스피커 클래스는 MusicPlayer 클래스의 상속을 받으면서 동시에 다른 기능들을 이용할 수 있는거죠. 참 신기합니다. 추상클래스든 인터페이스든 일단 다 안에는 추상메서드만 있으므로 전부 내용을 구체적으로 만들어주는 모습이 보이네요.

이제 이 클래스를 사용할 클래스를 만들어보겠습니다.

```
package com.koreait.music;

import com.koreait.ride.Roller;
import com.koreait.fly.Drone;
class UseSpeaker{

	public static void main(String[] args){
	
	BSpeaker bs = new BSpeaker();
	bs.supportMp3();
	bs.fly();
	bs.run();
	
	Drone d = bs; //스피커는 드론 형이다!
	d.fly();

	Roller r =bs;//스피커는 롤러 형이다!
	r.();

	}
}
```

BSpeaker 타입의 객체 bs를 만들고 이 객체로 부모클래스와 다른 인터페이스에 있는 메서드를 사용하는 모습을 볼 수 있습니다.

그리고 BSpeaker는 Drone형이나 Roller형으로 업캐스팅도 가능합니다!

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/1.png)

다 정상적으로 작동하는것을 볼 수 있습니다. ㅎㅎ 재밌네요.

이번엔 GUI에 대해서 알아보죠. gui에 대해선 할게 많지만 그래도 포스팅을 한 이유는 같이 알아보면서 javadoc을 100%활용하는 법도 알려드리도록 하겠습니다.

```
/*
윈도우 안에 버튼을 하나 넣어보기
자바에서 gui관련된 객체들은 java.awt 패키지에서 지원함..
*/
package com.koreait.gui;

import java.awt.*;

class GuiTest1{

	public static void main(String[] args){

	Frame frame= new Frame(); //gui에서 윈도우 역할을 하는 객체
	//윈도우창은 기본이 즉 default가 보이지 않는 속성이 적용됨
	frame.setVisible(true);//부모님 window클래스로부터 물려받은, 보이게 하는 메서드!
	frame.setSize(300,400);

	FlowLayout flow = new FlowLayout();
	frame.setLayout(flow);//프레임 안쪽의 배치방법을 결정 짓자!!
	//버튼을 선언하자
	Button bt= new Button("나 버튼");
	TextField t_id = new TextField(20);//입력 텍스트 컴포넌트
	Checkbox ch1 = new Checkbox("딸기",true);//체크박스
	Choice ch = new Choice();//select박스
	TextArea area= new TextArea(7,20);

	ch.add("이메일 선택");
	ch.add("google.com");
	ch.add("naver.com");
	ch.add("daum.net");

	frame.add(bt);//생성된 버튼을 프레임에 부착하자!!
	frame.add(t_id);//생성된 텍스트필드를 프레임에 부착
	frame.add(ch1);
	frame.add(ch);
	frame.add(area);

	}
}
```

자바에서 gui관련된 객체들은 java.awt패키지에 다 들어있습니다. 그래서 import로 import java.awt.\*;이게 보이죠 ㅎㅎ 이것은 awt의 하위들을 전부 임포트하겠다는 것입니다. 그래서 앞으로 쓸 버튼이라던지 체크박스라던지 전부 쓸 수 있습니다.

먼저 가장 큰 틀을 보겠습니다. 이것은 frame입니다. 근데 frame이 뭐냐고요? 저도 사실 잘 모릅니다. 그래서 우린 javadoc를 사용하죠.

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/2.png)

java 사이트에서 제공하는 8버전 javadoc입니다. 여기에서 우리가 평소에 쓰는 것부터 평생 살면서 한번이나 볼까한 것들을 다 찾아볼 수 있습니다. 일단 우리는 frame이라는 클래스를 알아야하니 java.awt에서 밑으로 내려서 frame을 찾아서 클릭한 화면입니다. 개요를 보시면 누구의 부모클래스를 상속받았는지 나옵니다. 가장 상위에는 Object가 있네요 .이 오브젝트는 마치 '물건'과도 같은 클래스라 모든 클래스가 여기로부터 파생된겁니다. 마치 헌법과도 같은 존재입니다.

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/3.png)

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/4.png)

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/5.png)

서머리에는 이 클래스가 어떤 생성자를 갖고 있는지, 어떤 필드가 있는지, 어떤 메서드를 갖고있는지 다 조회할 수 있습니다. 그래서 우리가 frame에 대해서 몰라도 생성자를 생성할 수 있고 여기서 필요한 메서드들을 호출할 수 있다는 거죠! javadoc은 앞으로 계속 참고해야할 중요한 웹페이지 입니다!

다시 저 클래스를 보자면 Frame타입의 객체 frame을 만들었습니다. 이것은 gui에서 윈도우창과 같습니다. 그런데 저것만 쓰면 막상 실행해도 보이지가 않는데요, 그래서 setVisible()메서드를 사용합니다. 사실 이것은 window클래스에 있는 frame클래스는 window클래스를 상속받았으니 사용할 수 있습니다. 내친김에 setVisible()메서드도 찾아볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/6.png)

이것은 Frame클래스를 클릭했을 때 밑으로 쭈욱 내리면 보이는 것들입니다. 여기는 Frame클래스엔 없지만 상속을 받아서 사용할 수 있는 메서드들을 전부 나타낸 것입니다. 그래서 부모님, 조부모님, 증조부모님들의 메서드들을 볼수있죠. 그래서 제가 형광펜을 칠한곳에 setVisible이 있네요. 들어가 봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/7.png)

메서드를 보면 매개변수에 boolean타입이 있군요. 내용을 대충 읽어보면 이것을 true로 했을 때 보이네요, 그래서 true로 매개변수를 넣어준 모습입니다.

setSize()는 말그대로 윈도우 창의 크기를 설정하는 것입니다. 그래서 마찬가지로 javadoc으로 확인한 뒤 매개변수에 적절한 값을 넣었습니다.

그리고 그 밑에는

TextField, Button, Checkbox, Choice, TextArea를 선언 한뒤 frame.add()로 다 추가해주는 모습입니다.

```
	ch.add("이메일 선택");
	ch.add("google.com");
	ch.add("naver.com");
	ch.add("daum.net");
```

이것은 choice 박스에 이메일 선택이란 제목으로 누르면 쭈욱 저것들이 나오죠. 한번 실행해 볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Abstract-Interface-GUI/8.png)

제대로 나오네요. 버튼과 텍스트 컴포넌트, 체크박스, choice리스트와, textarea가 나옵니다 ^^

이번 시간에는 여기까지 하겠습니다~