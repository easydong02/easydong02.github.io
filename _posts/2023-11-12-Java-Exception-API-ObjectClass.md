---
title: 예외처리 응용, API, Object클래스
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


이번 시간에는 저번에 개념만 알아봤던 예외처리에 대해 응용문제를 풀어보도록 하고 API, 그리고 Object클래스에 대해서 조금 알아보고 마치겠습니다.

문제는

무한 반복 안에서 5개의 정수만 입력받고 5칸 배열에 넣는 것입니다. 중요한 것은 try ~ catch문만 사용해서 문제를 풀어볼게요.

```
public class ExceptionTask {
	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		int[] arData = new int[5];
		int tmp=0;
		String temp =null;
		while(true) {
			System.out.println(++tmp+"번째 정수 입력[나가기 : q]: ");
			try {
				temp = sc.next();
				if(temp.equals("q")) {break;}
				arData[tmp-1]= Integer.parseInt(temp);
				System.out.println("추가 성공");
			} catch (NumberFormatException e) {
				System.out.println("정수만 입력해주세요.");
				tmp--;
			}catch(ArrayIndexOutOfBoundsException e) {
				System.out.println("더 이상 정수를 입력할 수 없습니다.");
				tmp--;
			}
			catch(Exception e) {
				System.out.println(e);
			}
		}
	}
}
```

try catch문으로 문제를 풀어보았는데요.. 사실 중간에 과정이 있습니다. 예외를 하나씩 알아내고 거기에 대해서 처리를 하는 과정이 있거든요.

먼저 5칸 배열에 입력 받을거니까 스캐너 클래스를 선언해주는 모습, 그리고 유틸리티 임포트까지 해줍시다. 그리고 5칸 짜리 정수타입 배열을 선언했습니다. 그리고 try문에는 예외가 일어날 것같은 문장을 적습니다. 이 문제에선 배열에 정수를 넣는 구문이겠죠? 그리고

arData\[tmp-1\]= sc.nextInt(); 대신 arData\[tmp-1\]= Integer.parseInt(temp); 를 쓴 이유는 만약 사용자가 정수가 아닌 다른 타입을 입력했을 때 정수 배열에 다른 타입을 계속 넣으려는 무한 루프를 없애기 위함입니다. 따라서 일단 배열속에 문자열이 들어가는 것까진 허용한 뒤에 당연히 오류가 나오겠죠? 그 오류가 NumberFormatException타입입니다. 따라서 catch문에서 처리하는 모습을 볼 수 있습니다. 그리고 tmp라는 장치를 만들어서 만약 정수가 들어가지 않았다면 -1하여 배열의 칸을 건너뛰는 일을 방지했습니다.

두번째 생길 수 있는 문제는 배열의 길이 인데요, 이 배열은 5칸까지 구현이 되어있습니다. 따라서 6번째 정수, 즉 배열의 인덱스5부턴 입력을 할 수 없죠. 따라서 이 문제에 대해서도 catch문으로 처리해보겠습니다. 일단 그 예외가 어떤이름인줄 모르니 Exception 타입으로 6번째 정수를 입력하면 ArrayIndexOutOfBoundsException라는 오류가 콘솔에 나옵니다. 따라서 catch문을 더 만들어서 이 예외를 처리하도록 하였습니다.

**API(Application Programming Interface)**

선배 개발자들이 미리 제작해놓은 소스코드들의 집합

\- 내부 API

JDK 설치 시 제공해주는 기본 API입니다.

docs.oracle.com/javase 여기에 들어가시면 우리가 평소에 쓰던 String이라던지 Class 등 기본적인 소스코드를 볼 수 있습니다.

\- 외부 API

업체 또는 기업 등의 선배 개발자들이 개발한 패키지 및 클래스들을 의미합니다.

보통 JAR파일로 배포하며 자바 프로젝트의 경로에 추가하여 사용할 수 있습니다.

\- 외부 API Build path에 추가

배포된 JAR 파일 다운 받기

\> 프로젝트 우클릭 > Build Path > Configure Build Path

\> Libraries 탭 클릭 > Add External JARs 클릭

\> 저장된 경로의 .jar파일 더블 클릭 > Apply 클릭

\> Orders and Exports 탭 클릭

\> Select all 클릭 > Apply and Close 클릭

이제 한번 외부에서 받아오고 반대로 제가 만들것을 외부 API로 전환해보겠습니다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/1.png)

제가 calc라는 jar파일을 다운받았습니다. 이것을 제가 사용하는 이클립스에 외부 API적용을 해볼게요!

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/2.png)

day14라는 프로젝트에 우클릭을 하시면

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/3.png)

이런 창이 나옵니다. 여기서 Build Path를 누르면

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/4.png)

Configure build path가 나오고 이걸 눌러봅시다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/5.png)

이런 창이 나오네요. 원래는 아무것도 없지만 제가 전에 3개를 추가했네요 ㅎㅎ그리고 JRE System Library가 보일겁니다. 이건 우리가 이클립스를 설치했을 때 기본적으로 오는 소스코드의 집합입니다. JDK 8버전인 것을 확인할 수 있죠. 이제 오른쪽 에 나열된 버튼들을 보시면 그중에 위에서 2번째 Add External JARs..가 보이실겁니다.

클릭하면,

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/6.png)

이런 탐색기가 열리고 여기서 아까 다운 받았던 JARs 파일 calc를 열기 해줍시다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/7.png)

그러면 목록중에 calc.jar라는 파일이 추가됐을거에요. 이제 오른쪽 버튼에서 밑으로 쭉 보시면 Apply버튼이 있을겁니다. 클릭해주고

현재 우리가 보고있는 것은 Libraries탭인데요, 그 바로 오른쪽에 Order and Export 탭을 눌러줍시다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/8.png)

이런 창이 또 나오죠. 원래는 아무것도 체크되어있지 않지만 여기서 오른쪽탭에 Select All 버튼을 누르시면 모두 체크가 된답니다. 그리고 또 Apply버튼을 클릭합니다. 그리고 Apply and Close 버튼 누르시면 됩니다. 이제 이클립스를 살펴볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/9.png)

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/10.png)

day14패키지를 보시면 원래있는 JRE System Library라는 기본 API가 있고, 그담에 제일 밑에 Referenced Libraries보이시나요?여기에 우리가 추가한 calc를 볼 수있습니다. 이제 우리의 클래스를 만들어서 사용을 해보겠습니다.

```
package apiTest;

import api.Calc;

public class ApiTest {
	public static void main(String[] args) {
		Calc c = new Calc();
		c.add(10, 9);
	}
}
```

그런데 calc이 어떤건줄 알고 제가 사용했을까요? 일단 import api.Calc;로 클래스에 임포트 한담에 Calc 타입의 객체 c를 만들었습니다.

그리고 c.을 입력하는 순간

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/11.png)

이런 것이 뜹니다. 그럼 우리가 Calc안에 있는 메소드가 add라는 메소드라는 것을 알 수 있습니다. 그리고 void 타입에 매개변수는 정수 2개를 받네요, 설명을 읽어보면 두 정수의 덧셈이란 것을 알 수 있습니다. 그래서 제가 c.add(10,9)를 클래스에 사용하면

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/12.png)

19가 잘 출력이 되는군요 ^^ 재밌습니다.

이제 우리가 만들어서 jar파일로까지 만들어보겠습니다.

```
package apiTest;

/**
 * 
 * @author 신동희
 *<br>
 *계산기
 *<br>
 * @since JDK8
 */
public class Calc {

	/**
	 * 
	 * @param num1 : First integer to divide
	 * @param num2 : Last integer to divide
	 * @return integer value
	 * @throws ArithmeticException
	 */
	public int div(int num1, int num2) {
		return num1/num2;
	}
}
```

/\*\*

\*

\* @author 신동희

\*<br>

\*계산기

\*<br>

\* @since JDK8

\*/

이건 뭘까요? 바로 어노테이션 주석이라고 합니다. 외부로 파일을 전달할 때 받는 사람이 그 API정보를 알 수 있도록 해놓은 주석인데요, 사용법은 일반 주석이랑 살짝 다릅니다. /\*\* 한 다음에 엔터를 치면 저런식으로 쭉 나옵니다. author에는 만든사람. 그리고 <br>은 한줄 뛰기 입니다. 그리고 @since에는 날짜를 쓰는 것이 아닌 만든 프로그램 버젼을 적습니다. 저는 JDK 8버전이기때문에 JDK8을 적었습니다.

그리고 클래스 안에

public int div(int num1, int num2) {

return num1/num2;

라는 메소드를 만들었습니다. 두 수를 나누는 거죠.

뿐만아니라 이 클래스 바디에도 어노테이션 주석을 만들 수 있습니다.

/\*\*

\*

\* @param num1 : First integer to divide

\* @param num2 : Last integer to divide

\* @return integer value

\* @throws ArithmeticException

\*/

param num1 ,num2는 아까 우리가 외부에서 calc를 가져오고 add라는 메소드를 사용할 때 봤던 설명처럼 나오게 하기 위함입니다. 그리고 throws에는 ArithmeticException이란 예외를 넣었는데요, 이건 두 수를 나눌 때 0으로 나눴을 때 그 예외를 무시하기 위한 작업입니다.

이제 클래스를 다 만들었고 이걸 jar파일로 변환시켜볼게요!

클래스를 우클릭하면

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/13.png)

이런 창이 나오고 여기서 Export를 누릅니다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/14.png)

그리고 JAVA폴더에 JAR file을 누릅니다. 그리고 넥스트버튼 클릭

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/15.png)

여기에 보시면

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/16.png)

이 사진처럼 2개를 체크 해주고 밑에 파일경로에 원하는 경로를 설정한뒤 finish 버튼을 누르면 jar파일 형태로 됩니다.

**Object 클래스 : 최상위 부모 클래스**

1\. toString()

항상 객체명을 출력할 때에는 toStrng()이 생략되어 있습니다.

toString()을 통해 출력되는 문자열이 마음에 들지 않는다면, 재정의하여 수정하도록 합니다.

예를 들어,

```
package obj;

public class Student {
	int number;
	String name;
	int age;
	
	public Student() {;}

	public Student(int number, String name, int age) {
		super();
		this.number = number;
		this.name = name;
		this.age = age;
	}
	
	@Override
	public String toString() {
		String str= "번호: " + number
				+ "\n이름 : " + name
				+ "\n나이 : " + age + "살";
		return str;
	}
	
}
```

student라는 클래스를 만들고 기본 필드로 number, name, age를 넣습니다. 그리고 기본생성자와 직접생성자 2개를 만들죠. 그리고 중요한 것은 toString은 생략되어있다 했죠? 따라서 재정의로 return을 제가 만든 문자열로 했습니다.

그럼 다른 클래스르 볼게요.

```
package obj;

public class School {
	public static void main(String[] args) {
		Student 신동희 =new Student(1,"신동희",20);
		System.out.println(신동희);
	}
}
```

School이란 클래스를 만들고 신동희라는 객체를 만듭니다. 필드엔 1,"신동희",20을 넣었습니다. 이제 객체명만 출력문에 입력하면 그 필드의 주소값만 나오는데 아까 재정의 때문에

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/17.png)

이렇게 나옵니다! 신기하죠?

2\. equals() : 주소값 비교(==)

※ String 클래스에서는 값 비교로 재정의 되었습니다.

우리가 원래 비교할 때 '~와 같다' 라는 값의 비교를 ==으로 사용하고 있었죠? 하지만 이것은 사실 주소값비교였습니다. 말이 이상하죠?

```
String data1 = "ABC";
String data2 = "ABC";
System.out.println(data2);
```

자 이제 문자열을 살펴볼게요, data1,data2는 변수일까요? 아닙니다. String은 엄연히 클래스이기 때문에 data1,2는 객체라고 볼 수있죠. 그럼 그 객체만을 출력하면 주소값이 나와야 하는데 왜 값이 나올까요? 이건은 아까 배웠던 toString때문입니다. data1은 객체이기 때문에 우리가 초기화를 저런식으로 "ABC"라고 해도 사실 String data1 = new String("ABC");가 되는거죠. 아까 Student에서 신동희라는 객체를 썼을때 주소값이 안나오고 재정의한 값이 나왔죠? 마찬가지 입니다. new String("ABC")로 재정의 했기 때문에 우리가 바로 값을 본다라고 생각한거죠.

이제 자세히 살펴보죠.

```
package obj;

public class StringTest {
	public static void main(String[] args) {
		String data1 = "ABC";
		String data2 = "ABC";
		
		String data3= new String("ABC");
		String data4= new String("ABC");
		
		System.out.println(data1.toString());
		System.out.println(data2);
		System.out.println(data1==data2);
		System.out.println(data1.equals(data2));
		
		System.out.println(data3 ==data4);
		System.out.println(data3.equals(data4));
	}
}
```

data 1~4 까지 만들었습니다. 1,2는 일반적으로 String에 사용하는 방법이고 3,4는 클래스 생성자 방식으로 해봤어요.과연 그냥 ==와 equals. 의 차이는 무엇일까요?

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/18.png)

실행한 값입니다.

data1,2를 선언할 때는 일반적인 방식, 즉 '값'을 중점으로 선언했기 때문에 ==를 하면 true가 나오는 것이죠, 마찬가지로 data1.equals.(data2)를 해도 true가 나오고요. 하지만 data3,4를 선언할 때는 '주소값'을 중점으로 선언되었기 때문에 주소값 비교인 ==에서 false가 난 것이죠. 그럼 equals도 주소값 비교인데 왜 true가 나올까요? 이건 String 클래스를 자세히 봐야합니다. String에 컨트롤+클릭 하면 이 스트링의 코드가 쭉 나오는데요

우리가 무심코 썼던 String이란 클래스는 사실 3100여 가량의 코드 라인으로 이뤄진 엄청난 클래스입니다. 여기서 equals를 찾아볼게요.

```
public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

equals 메소드를 찾았습니다. 이 메소드의 매개변수는 Object네요? Object는 자바의 모든 클래스들의 부모타입입니다. 따라서 아무거나 해도 Object타입이면은 다 들어올 수 있죠. 그리고 if문이 보이네요, ==은 주소값 비교라고 했죠? 매개변수로 들어온 것의 주소값이 String과 같다면 당연히 같겠죠? 그럼 true를 리턴합니다. 이제 문제입니다. data3, data4처럼 주소값이 같지 않다면 일단 다음 if문에서 타입비교를 하는 것이 보이죠. 둘다 String 타입이면 통과 아니면 false를 리턴합니다. 그리고 value.length는 길이입니다. 길이가 같으면 통과 아니면 false를 리턴합니다. 그 다음은 이제 각 String의 문자값을 하나씩 비교합니다. 전부 같으면 드디어 true를 리턴하고 아니면 false를 리턴합니다. 따라서 아까 data3,4를 주소값비교인 equals를 해도 true가 나오는 이유입니다.

이번시간에는 정말 많은 내용을 담고있었습니다. 따라서 여러번 보면서 확실히 제것으로 만드는 작업이 필요하다고 생각했습니다.