---
title: [Java] 파일입출력 예외처리
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번 시간에는 파일입출력에 대해서 알아보고 간단한 이미지 다운로드 어플리케이션을 만들어보겠습니다 ~

이미지 파일을 실행중인 자바프로그램으로 읽어들인 후, 목적지 경로에 다시 출력해봅시다 =즉 파일 복사를 구현하는 작업입니다.

주제 : Stream(흘러가는 강 줄기 등) 컴퓨터 분야에서의 스트림의 대상은 바로 데이터! 자바에서는 스트림과 관련된 유용한 api를 java.io패키지에서 지원합니다!

**스트림의 분류**

1) 방향에 따른 분류 (기준은 실행중인 프로그램이다 - 상대적 개념)

입력: 실행중인 프로그램으로 데이터가 들어가는 것

출력: 실행중인 프로그램에서 데이터가 나오는 것

입출력에 필요한 클래스 3개를 일단 나열할게요(입력기준)

FileInputStream : 데이터를 byte단위로 한개씩 정성스럽게(?)옮기는 녀석입니다. 가장 근본적인 입출력 클래스라 배울게 많지만 성능은 느립니다.

FileReader : 데이터를 byte묶음 단위로 전송합니다. 유니코드가 호환되는 위 클래스의 업그레이드 버전입니다.

BufferedReader : 파일 입출력에서 가장 많이 쓰인다고 생각하시면 될거같습니다. 엔터(\\n)를 기준으로 그 전의 값을 유동적으로 가져올 수 있습니다.

```
package com.koreait.project0830.stream1;

import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;

class ImageFileCopy{
	FileInputStream fis;//파일을 대상으로 데이터를 읽어들이는 데 사용하는 스트림 객체
	FileOutputStream fos;//파일을 대상으로 데이터를 내보내기 위한 스트림 객체
	public ImageFileCopy(){


		try{
			fis = new FileInputStream("D:/bigData/workspaceOfJava2/data/gini3.jpeg");
			fos = new FileOutputStream("D:/bigData/workspaceOfJava2/data/gini1.jpeg");	//empty 파일 생성해줌
			System.out.println("스트림 생성 성공");
			
			int data=-1;

			while(true){
				data = fis.read();//1byte 읽어들임
				if(data==-1)break;//파일의 마지막에 도달한 경우엔 반복문 빠져나오기
				//System.out.println(data);//읽어들인 데이터 1byte 출력
			
				fos.write(data);//빈 파일을 대상으로, 1byte알갱이를 내뱉자!
			}
		}catch(FileNotFoundException e){

			System.out.println("해당 파일을 찾을 수 없습니다.");
			e.printStackTrace();
		}catch(IOException e){
			System.out.println("파일 읽기에 실패했습니다.");
			e.printStackTrace();
		}finally{
			//메모리 올라온 경우만..
			if(fis !=null){
				try{
				fis.close();//null 객체는 메서드 호출 불가능..NullPointerException
				}catch(IOException e){
					e.printStackTrace();
				 } 
			}
			if(fos !=null){
				try{
				fos.close();
				}catch(IOException e){
					e.printStackTrace();
				 }
			}
		 }
	}
	public static void main(String[] args) {
		new ImageFileCopy();
	}
}
```

FileInputStream예제 클래스입니다. 그런데 여기서 생소한 문법이 보이시죠? try~ 어쩌구 catch어쩌구 적혀있는데 이 예외처리에 대해서 한번 알아보도록 하겠습니다

**예외처리**

프로그램의 문법이 문제가 아닌, 외부적 문제로 인한 문제가 발생할 경우 프로그램은 어떻게 될까요?비정상 종료에 이르게 된다.. 컴퓨터 분야에서 가장 무서워 하는 상황이라고 합니다.. 왜냐면 코드적인 문제가 아니라 어떻게 할 수 없는 외부적인 요인이기 때문이죠. 따라서 보안 안정적인 프로그램의 실행을 위해 예외처리라는 기술을 도입했습니다.

결론 : 문법적으로 문제가 없다하여, 프로그램이 무사히 수행될 것이라는 보장을 할 수 없습니다.. 이런 예외적인 상황에 프로그램을 안정적으로 수행시키기 위한 방법이 바로 예외처리 인데요

바로 try~catch 문으로 지원합니다.

실제로 해당 예외가 발생하면, 예외를 객체의 인스턴스를 메모리에 올리고 e로 전달합니다. 그리고 개발자는 전달받은 e를 이용하여 원인을 분석, 로그기록 등을 처리합니다. 원래는 비정상종료 되었어야 하는 프로그램의 실행부를 catch문으로 진입시켜 안정화시키는 것이죠. 원상복구는 불가능하여, 주로 에러의 원인 등 로그를 남기거나, 이메일 ,기타 관리자에게 피드백 처리를 합니다.

결론: 예외처리의 목적은 프로그램의 비정상 종료의 방지!!

이제 이론 설명은 여기까지하고 위의 클래스를 분석해볼게요!

```
fis = new FileInputStream("D:/bigData/workspaceOfJava2/data/gini3.jpeg");
fos = new FileOutputStream("D:/bigData/workspaceOfJava2/data/gini1.jpeg");	//empty 파일 생성해줌
```

먼저 각 FileInputStream과 FileOutputStream의 객체 fis,fos를 생성해주었습니다. fis에는 로컬 디렉토리에 있는 파일 경로를 가져와서 담아줍니다. 그리고 fos는 매개변수 안에 있는 경로로 빈 파일을 생성해줍니다. 이제 앞으로 여기에 fis의 데이터가 들어와서 복사가 되는 것이죠.

```
			while(true){
				data = fis.read();//1byte 읽어들임
				if(data==-1)break;//파일의 마지막에 도달한 경우엔 반복문 빠져나오기
				//System.out.println(data);//읽어들인 데이터 1byte 출력
			
				fos.write(data);//빈 파일을 대상으로, 1byte알갱이를 내뱉자!
            }
```

FileInputStream에겐 read()라는 메서드가 있습니다. 이것은 아까 말씀드린 것처럼 1byte씩 자료를 가져오는 것이죠. 그리고 그것을 정수타입의 data에 넣습니다. 그리고 무한루프 안에 넣어준 뒤 조건문으로 data가 -1일때 빠져나오도록 했습니다. 왜 -1일까요? read()메서드는 더이상 자료가 없으면 -1을 반환하기 때문입니다. 마찬가지로 이렇게 들어온 데이터를 FileOutputStream의 메서드 write()로 아까 로컬 디렉토리에 적어놓은 경로에 있는 빈파일에 하나씩 옮깁니다! 그래서 데이터가 채워지는 것이죠.

```
finally{
			//메모리 올라온 경우만..
			if(fis !=null){
				try{
				fis.close();//null 객체는 메서드 호출 불가능..NullPointerException
				}catch(IOException e){
					e.printStackTrace();
				 } 
			}
			if(fos !=null){
				try{
				fos.close();
				}catch(IOException e){
					e.printStackTrace();
				 }
			}
		 }
```

finally는 마치 switch문에서 있는 default문같이 마지막에 무조건 실행되는 바디입니다. 즉 예외가 발생해도 실행되는 부분입니다. 파일 입출력은 아까 데이터를 옮기고 끝이 아니라 close()메서드로 매듭을 지어줘야 합니다. 그런데 예외가 발생해도 finally부분은 실행 됩니다.그래서 만약 입력에서 경로를 잘못 설정해서 객체 fis에 아무값이 없다면 close()메서드에도 문제가 생기기 때문에 조건문으로 null값이 아닐때, 즉 제대로 데이터가 들어갔다가 나왔을 때만 close()메서드를 실행할 수 있도록 설정하였습니다.

![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/1.png)

파일 경로에 잘 생성되었습니다 ㅎㅎ 제가 통통 성애자라 기니피그를 좋아합니다.. 나중에 키우고 싶어요..

```
package com.koreait.project0830.stream3;

import java.io.FileReader;
import java.io.FileWriter;
import java.io.FileNotFoundException;
import java.io.IOException;

class  ReadCharacter {
	FileReader reader; //파일을 읽고, 그 읽은 데이터를 2byte씩 묶어 문자로 이해하는 스트림
	FileWriter	writer;//문자를 출력할 수 있는 스트림
	
	//WEB개발자 MVC패턴 --> 모듈 별로 분류가 됨... 즉 역할별로 프로그램을 분류..
	//throws의 의미: 개발자가 처리할 예외를, 현재의 메서드 호출부에게 전가시키는 방법..
	public ReadCharacter() throws FileNotFoundException,IOException{
		reader = new FileReader("D:/bigData/workspaceOfJava2/data/memo.txt");
		writer = new FileWriter("D:/bigData/workspaceOfJava2/data/memo_copy.txt");
	
		//한 문자 읽기!
		int data=-1;
		while(true){
			data = reader.read();
			if(data==-1)break;
			writer.write(data);
		}
		if(reader !=null){reader.close();}
		if(writer !=null){writer.close();}
	}

	public static void main(String[] args) throws FileNotFoundException,IOException{//jvm...
		new ReadCharacter();
	}
}
```

FileInputStream은 read()메서드에 의해 읽어들이는 데이터가 1byte이며,2byte를 하나의 문자로 읽을 수 있는 능력이 없습니다.. 따라서 영문의 경우엔 1byte차지 하므로, 읽어들인 데이터를 문자로 변경하는데 문제가 없지만, 한글의 경우엔 유니코드 기반이라 2byte로 하나의 문자를 표현하기 때문에 read()메서드로 읽어들인 1byte는 한글을 표현할 수 없습니다. 해결책은? 읽어들인 데이터를 대상으로 2byte씩 묶어서 문자로 이해하는 업그레이드 된 문자기반 스트림을 이용해야 합니다.

이번에는 한글이 포함되어있는 txt파일을 복사하겠습니다. 문법은 같지만 이것은 2바이트인 한글뿐만 아니라 여러 제3외국어도 사용가능합니다! 여기서 다른점은 아까는 모두 try~catch문으로 일일이 적어주었다면 이번엔 클래스명 뒤에 throws를 통해서 예외를 다른 데로 전가시키는 것입니다. 능력은 트라이캐치랑 비슷하지만 트라이캐치는 각 예외에 대해서 어떻게 반응할지 개발자가 정해줄 수 있지만 이것은 단어 그대로 던져버립니다... 실행하면 무사히 텍스트파일이 복사된 것을 알 수 있습니다.

이제 다른 입력에서도 알아볼까요? 우리가 출력문을 쓸 때 무심코 쓴 System.out.println도 사실 입출력의 한부분입니다.

이 System이란 클래스는 근본적인 클래스이며 여기에는 딱 3개! err, out, in이 있습니다. out은 표준 출력이고 in은 표준 입력입니다. 이 System.in으로 우리가 키보드로 입력한것을 할 수 있죠.

```
InputStream is =System.in;//표준입력스트림, 주로 키보드 등에 연결된 스트림..
		int data=-1;

		while(true){
			data = is.read();//키보드로 부터 1byte 읽어들임
			System.out.print((char)data);
```

이렇게 하면 콘솔창에서 입력한대로 됩니다 !

이제 이것을 응용해서 인터넷 이미지 주소를 원하는 경로에 다운받는 프로그램을 만들어보겠습니다!

```
package com.koreait.project0830.stream7;

import java.awt.Dimension;
import java.awt.FlowLayout;
import java.awt.Font;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
import javax.swing.JTextField;

import com.koreait.common.FileManager;

public class DownLoader extends JFrame implements ActionListener {
	JTextField t_url;// 이미지 경로
	JButton bt;
	HttpURLConnection httpCon; // 웹상의 요청을 시도하는데 사용하는 객체(브라우저 없이 자바코드에서)
	URL url;
	FileOutputStream fos;
	InputStream is;

	public DownLoader() {
		// 생성
		t_url = new JTextField();
		bt = new JButton("DownLoad");

		// 스타일
		this.setLayout(new FlowLayout());
		t_url.setPreferredSize(new Dimension(570, 40));
		t_url.setFont(new Font("Verdana", Font.BOLD, 20));

		// 조립
		add(t_url);
		add(bt);

		// 윈도우 설정
		setSize(600, 120);
		setVisible(true);
		setDefaultCloseOperation(EXIT_ON_CLOSE);

		// 연결
		bt.addActionListener(this);
	}
	public static void main(String[] args) {
		new DownLoader();
	}

	// 이미지 다운로드
	public void getImage() {
		// 스트림을 생성하되, 로컬상의 파일이 아닌, 원격지의 url로부터 스트림을 생성..
		try {
			url = new URL(t_url.getText());// 사용자가 입력한 값을 url의 생성자 매개변수로 활용
			URLConnection con = url.openConnection();
			httpCon = (HttpURLConnection) con;
			// 이시점(httpcon메모리 올린)부터, 웹상의 필요한 자원에 스트림 연결이 가능..
			is = httpCon.getInputStream();
			// 사용자가 입력한 텍스트필드의 값중에서 확장자만 추출
			String ext =FileManager.getExt(t_url.getText());
			fos = new FileOutputStream("D:\\bigData\\workspaceOfJava2\\data\\ginigini"+ext);// 저장할 파일명..
			// 읽기
			int data = -1;
			while (true) {
				data = is.read();
				if (data == -1) {
					break;
				}
				System.out.println(data);
				fos.write(data);
			}

			JOptionPane.showMessageDialog(this, "읽기 완료");
			// 열려있는 스트림은 닫아야 한다!!

		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (is != null) {
				try {
					is.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (fos != null) {
				try {
					fos.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}

	}
	@Override
	public void actionPerformed(ActionEvent e) {
		getImage();
	}
}
```

길어보이지만 하나의 메서드만 보면 됩니다. 바로 getImage()메서드입니다. 일단 JFrame으로 텍스트 필드와 버튼을 만들어서 텍스트필드에 이미지 주소값(http:// ~~.jpg)을 적고 버튼을 누르면 미리 지정한 경로에 파일을 복사해서 만들어주는 겁니다. 그래서 버튼을 누르면 getImage()메서드가 실행되는데 이부분을 볼게요!

```
	// 이미지 다운로드
	public void getImage() {
		// 스트림을 생성하되, 로컬상의 파일이 아닌, 원격지의 url로부터 스트림을 생성..
		try {
			url = new URL(t_url.getText());// 사용자가 입력한 값을 url의 생성자 매개변수로 활용
			URLConnection con = url.openConnection();
			httpCon = (HttpURLConnection) con;
			// 이시점(httpcon메모리 올린)부터, 웹상의 필요한 자원에 스트림 연결이 가능..
			is = httpCon.getInputStream();
			// 사용자가 입력한 텍스트필드의 값중에서 확장자만 추출
			String ext =FileManager.getExt(t_url.getText());
			fos = new FileOutputStream("D:\\bigData\\workspaceOfJava2\\data\\ginigini"+ext);// 저장할 파일명..
			// 읽기
			int data = -1;
			while (true) {
				data = is.read();
				if (data == -1) {
					break;
				}
				System.out.println(data);
				fos.write(data);
			}

			JOptionPane.showMessageDialog(this, "읽기 완료");
			// 열려있는 스트림은 닫아야 한다!!

		} catch (MalformedURLException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (is != null) {
				try {
					is.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (fos != null) {
				try {
					fos.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}

	}
```

일단 Try문을 볼게요. 우리는 실질적으로 네트워크의 주소를 가져올겁니다. 그래서 Url객체로 접근을 해야하죠. 이것을 우리가 직접 갖고 놀 수 있는 Stream타입으로 바꾸어야합니다. 먼저 텍스트필드 t\_url.getText()를 통해서 그 안에 있는 문자열을 다 가져오고 이것을 Url타입으로 객체를 만듭니다. 그리고 이것을 다시 URLConnection타입으로 바꿔준뒤 그 자식인 HttpURLConnection타입으로 다운캐스팅했습니다. 뭐가 빠르죠? ㅎㅎ; 그냥 계속 단계별로 형변환을 그 클래스에 있는 메서드를 사용해서 해준겁니다. 그리고 HttpURLConnection 클래스에 있는 getInputStream()메서드로 최종적으로 is라는 객체에 담아주었습니다. 그리고 getExt()메서드가 보이네요. 이것은 그냥 아까 넣은 주소값에서 마지막 확장자를 문자열로 가져오는 기교부리는 메서드라 크게 중요하진 않습니다. 그래서 is에 네트워크에 있는 이미지파일의 데이터를 가져오는데 성공했습니다. 이제 이것은 아까 배운 FileOutputStream클래스로 우리가 원하는 경로에 복사만 해주면 됩니다! 실행해볼까요?

![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/2.png)

이렇게 인터넷에 있는 한 이미지 파일의 주소를 복사하고 텍스트 필드에 넣은뒤 버튼을 누르면?

![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/3.png)

이렇게 잘 나옵니다 ^^

이번 시간엔 여기까지 하겠습니다!