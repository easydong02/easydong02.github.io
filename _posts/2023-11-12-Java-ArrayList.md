---
title: "[Java] ArrayList"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


안녕하세요! 이번시간에는 ArrrayList를 활용한 응용문제를 풀어보겠습니다!

먼저 데이터들을 모을 DB를 만들고, 아이디는 중복이 되면 안 되니까 중복검사도 넣어야겠죠?중복검사는 로그인과 회원가입 두 군데에서 모두 쓰일겁니다. 그리고 비밀번호를 암호화한다음 다시 DB에 대입해주고 실행까지 해볼게요~

**Class1. User**

User클래스를 먼저 선언해보도록 할게요~

```
package day16;

public class User {
	private String id;
	private String pw;
	
	public User() {;}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getPw() {
		return pw;
	}

	public void setPw(String pw) {
		this.pw = pw;
	}
}
```

일단 회원가입과 로그인을 할거라 id와 pw만 만들어주고 기본생성자와 게터세터를 만들어서 private 필드들에 대한 수정과 호출을 할 수 있게 해주었습니다.

**Class2. UserField**

**//DB**

```
//DB
	ArrayList<User> users = new ArrayList<>();
	Scanner sc = new Scanner(System.in);
```

User타입으로 각 객체의 주소값을 저장하고 있는 배열을 만들었습니다. 일반 배열이 아닌 ArrayList로 만든 것은 당연히 추가,수정,삭제 등이 편하기 때문이겠죠?

**//아이디 중복검사**

```
public User checkId(String id) {
			User user = null;
		for (int i = 0; i < users.size(); i++) {
			if(users.get(i).getId().equals(id)) {
				user= users.get(i);
			}
		}
		return user;
	}
```

중복검사기능을하는 checkId필드를 만들었습니다. 문자열 id를 매개변수로 하고요 리턴타입은 User입니다. for문으로 ArrayList안에 있는 객체와 매개변수로 입력된 id가 같을 경우(중복인 경우) User타입의 객체 user에 원래 있던 id값을 넣어주고 리턴합니다. 매개변수로 입력된 id가 if문 조건에 만족하지 않는다면(중복이 없을 경우) user에 null이라는 초기값이 그대로 리턴이 됩니다.

**//암호화**

```
	public String encrypt(String pw) {
		String en_pw ="";
		
		for (int i = 0; i < pw.length(); i++) {
			en_pw +=(char)pw.charAt(i)*3;
			
		}
		
		return en_pw;
	}
```

pw를 그냥 값으로 넣으면 보안위험이 있을 수 있기 때문에 간단하게 암호화를 진행시키겠습니다. 단방향으로 진행했습니다. String타입의 메소드이구요 매개변수로는 문자열을 입력받습니다. 그리고 en\_pw에 기존의 pw를 아스키코드 x3으로 넣었습니다. 그리고 그 값을 리턴하죠.

**//회원가입**

```
public void makeId(User user) {
		user.setPw(encrypt(user.getPw()));
		users.add(user);
		
	}
```

회원가입은 은근히 간단하죠? 처음 만들 때 여기에 중복검사도하고 이것저것 넣으려고 했지만, 메소드 시간에 배운 것은 모든 기능을 한 메소드에 넣지말고 특정성도 주지말자! 라는 것입니다. 기억하시죠?ㅎㅎ 그래서 간단히 아이디와 패스워드를 add함수로 넣었습니다. 패스워드는 아까 만든 암호화 메소드를 활용해서 암호화된 값을 넣었습니다!

**//로그인**

```
public  boolean logIn(String id, String pw) {
		User user = checkId(id);
		if(user != null) {
			if(user.getPw().equals(encrypt(pw))) {
				return true;
			}
		}
		return false;
	}
```

로그인입니다. 메소드의 리턴타입은 boolean으로 했습니다. 매개변수에 아이디와 패스워드를 넣은 후 중복검사 메소드로 id를 중복검사합니다. 로그인할 때는 중복이어야 진행이 가능하겠죠? 그래서 조건문으로 이 메소드의 값이 null이 아닐 때, 그리고 입력한 패스워드를 암호화했을 때 이게 기존에 있던 값이랑 같으면 true를 리턴하고 그렇지 않으면 false를 리턴하도록 했습니다.

**Class3. UserMain**

우리가 실제로 사용할 클래스입니다~ 만든 것을 써보도록 할게요!

```
package day16;

public class UserMain {
	public static void main(String[] args){
		UserField uf = new UserField();
		User newUser = new User();

		User checkUser = uf.checkId("easydong02");
		
		if(checkUser == null){
			System.out.println("사용 가능한 아이디");
			newUser.setId("easydong02");
			newUser.setPw("1234");
			uf.makeId(newUser);
			System.out.println("회원가입 성공!");
		}else{
			System.out.println("중복된 아이디");
		}		

		checkUser = uf.checkId("easydong02");
		
		if(checkUser == null){
			System.out.println("사용 가능한 아이디");
		}else{
			System.out.println("중복된 아이디");
		}
	}
}
```

통채로 옮겼습니다! id를 한번 만들어보고 잘 만들어지는지, 만들어진 id와 같은 id를 만들었을 때 중복검사 메소드가 잘 작동하는지 볼게요

먼저 UserField클래스의 객체 uf와 새로만들 id를 사용할 User클래스의 객체 newUser 를 만들었습니다. 그리고 User타입의 객체 checkUser를 만들어서 중복검사를 해주겠습니다.

먼저 easydong02라는 아이디가 있는지 검사하고 없으면 checkUser에 easydong02를 넣었습니다. 물론 처음 만든 ArrayList라 안에 값은 아무것도 없으니 checkUser에 저 아이디가 들어가겠죠 ㅎㅎ 참고로 easydong02는 이 네이버 아이디입니다 ㅎ..

그리고 if문으로 갑시다. 조건문에 checkUser가 null인지 아닌지 검사를합니다. checkId메소드를 통해서 easydong02라는 아이디가 들어갔으므로 if 문 바디를 실행시키겠죠? 잘 들어왔으니 "사용 가능한 아이디"라는 문자열을 출력합니다. 그리고 아까 만든 newUser객체에 이 아이디와 패스워드를 넣어줍니다. 그리고 add메소드로 이 객체를 ArrayList에 추가했습니다. 그리고 "회원가입 성공!"이라는 문자열을 출력하도록 했습니다. 그렇지 않으면 "중복된 아이디"라는 문자열을 출력하도록 했습니다.

이제 easydong02과 1234라는 패스워드가 암호화가 되어 회원가입이 되었을 테니 똑같은 아이디를 생성시도하여 중복검사메소드가 제대로 실행되는지 보겠습니다. 아까 만든 checkUser객체에 다시한번"easydong02"를 매개변수로하는 checkId메소드의 리턴값을 넣어줍니다. 이제 checkUser에는 null값이 들어가겠죠? 이제 같은 if문을 만듭니다. 이번에는 조건문을 통과하지 못하므로 "중복된 아이디"가 출력되겠네요.

따라서 이 클래스의 메인메소드를 실행시키면

"사용 가능한 아이디"

"회원가입 성공!"

"중복된 아이디"

라는 3문장이 출력이 되겠네요.

![Desktop View](/assets/img/Programming-Language/Java/ArrayList/1.png)

잘 출력이 되었습니다.

처음 할 때는 회원가입과 로그인? 엄청 어려운거 아니야? 라고 생각했지만 실제로 만들어보니 배웠던 것들로 충분히 할 수 있는 문제였습니다. 물론 실무에서는 훨씬 더 많은 기능(본인인증, 패스워드 규칙, 주소입력 등)을 넣어야 하겠지만 그 때도 지금처럼 차근차근 만들다 보면 좋은 툴을 만들 수 있지 않을까 싶습니다. 그럼 이번 포스팅을 마치겠습니다.