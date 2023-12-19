---
title: "[Java] Java GUI로 도서어플 만들기"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---
이번엔 도서관리 프로그램 (알라딘 같은?) 어플리케이션을 만들어보았습니다.. 너무 힘들었습니다..ㅜ.ㅠ 그래도 하나씩 배우니까 이것만 잘 봐도 실력 많이 늘겠다 싶어서 긍정적인 마인드 장착하고 수업 잘 들었습니다 ㅎㅎ 아직 다 완성 된것은 아니지만 차차 완성시키면서 배운 것과 익힌 것을 잘 적어보도록 할게요! 일단 MVC원리에 의해서 패키지는 view와 model로 나눠서 view는 디자인 담당하는 클래스들만 넣었고 model은 로직을 담당하는 클래스만 넣어서 추후에 유지보수가 편하도록 짰습니다!

//MainFrame

```
//어플리케이션의 메인 윈도우 (View == 디자인 )
public class MainFrame extends JFrame implements ActionListener {
	JPanel p_north;
	JPanel p_center;// 모든 페이지가 공존할 컨테이너 역할
	JButton bt_main; // 도서관리 메인 페이지
	JButton bt_schedule; // 일정관리
	JButton bt_regist; // 회원가입
	JButton bt_cs; // 고객센터
	JButton bt_login; // 로그인 or 로그아웃

	BookMain bookMain;// 도서관리 페이지
	Schedule schedule;
	MemberRegister memberRegister;
	CSMain csMain;
	LoginForm loginForm;

	ArrayList<JButton> btnList;// 버튼을 가리킴에 있어 규칙을 만들기 위해
	ArrayList<Page> pageList;// 페이지를 가리킴에 있어 규칙을 만들기 위해

	// 로그인 상태 여부...
	Admin loginObj; // null

	public MainFrame() {
		// 생성
		p_north = new JPanel();
		p_center = new JPanel();
		bt_main = new JButton("도서관리메인");
		bt_schedule = new JButton("일정관리");
		bt_regist = new JButton("회원가입");
		bt_cs = new JButton("고객센터");
		bt_login = new JButton("Login");

		bookMain = new BookMain(this);
		schedule = new Schedule(this);
		memberRegister = new MemberRegister(this);
		csMain = new CSMain(this);
		loginForm = new LoginForm(this);

		btnList = new ArrayList<JButton>();
		pageList = new ArrayList();

		// 부모인 Page객체에 mainFrame을 전달하자
		// memberRegister.setMainFrame(this);//매개변수로 설정해서 필요없음.

		// 버튼을 개성있는 이름이 아닌, 규칙있는 숫자로 가리키기 위해
		btnList.add(bt_main);
		btnList.add(bt_schedule);
		btnList.add(bt_regist);
		btnList.add(bt_cs);
		btnList.add(bt_login);

		// 페이지들을 개성있는 이름이 아닌, 규칙있는 숫자로 가리키기 위해
		pageList.add(bookMain);
		pageList.add(schedule);
		pageList.add(memberRegister);
		pageList.add(csMain);
		pageList.add(loginForm);

		// 스타일
		p_north.setBackground(Color.GRAY);
		p_center.setBackground(Color.BLACK);

		// 조립
		p_north.add(bt_main);
		p_north.add(bt_schedule);
		p_north.add(bt_regist);
		p_north.add(bt_cs);
		p_north.add(bt_login);
		add(p_north, BorderLayout.NORTH);
		p_center.add(bookMain);
		p_center.add(schedule);
		p_center.add(memberRegister);
		p_center.add(csMain);
		p_center.add(loginForm);

		// 프레임에 페이지 부착
		add(p_center);

		// 윈도우 설정
		setSize(1200, 800);
		setVisible(true);
		setDefaultCloseOperation(EXIT_ON_CLOSE);
		setLocationRelativeTo(null);

		// 리스너와 연결
		for (JButton jbtn : btnList) {
			jbtn.addActionListener(this);
		}
		// 초기화면 설정
		showHide(4);
	}

	// 페이지 보이고, 안 보이게 처리 n의 값은 visible을 true로 놓을 대상 페이지
	public void showHide(int n) {
		for (int i = 0; i < pageList.size(); i++) {
			Page page = pageList.get(i);
			if (i == n) {
				page.setVisible(true);
			} else {
				page.setVisible(false);
			}
		}
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		JButton btn = (JButton) e.getSource();

		int index = btnList.indexOf(btn);
		if (loginObj == null) {
			if (index == 0 || index == 1 || index == 3) {
				JOptionPane.showMessageDialog(this, "로그인이 필요한 서비스입니다.");
			} else { // 2,4
				showHide(index);// 어떤 버튼을 누르면 그 인덱스 추출해서 매개변수로 전달
			}
		} else {
			// 로그인한 상태에서 만일 로그아웃 버튼 누르면?
			if (index == 4) {// 로그아웃 대상
				showHide(index);
				loginObj = null;
				JOptionPane.showMessageDialog(this, "로그아웃 되었습니다.");
				bt_login.setText("Login");
			} else {
				showHide(index);
			}
		}
	}

	public static void main(String[] args) {
		new MainFrame();
	}

}
```

메인프레임입니다. 여기서 모든것들이 조립되고 실행됩니다. 보시다시피 임포트문도 많고 선언분도 엄청 많습니다. 저는 이번에 프로그램에 한 화면만 띄워져 있는 것이 아니라 여러 탭을 만들고 그것을 누르면 화면을 바꿀 수 있게 하였습니다! 따라서 나중에 액션 리스너가 인식하기 위해 각 버튼과 연결된 페이지가 인덱스를 갖고 있어야 하겠죠? 그래서 btnList와 pageList를 어레이리스트 타입으로 만들었습니다. 그리고 나중에 로그인을 확인할 수 있는 loginObj도 만들었습니다! 보시면 생성부분에서 각각의 파트들이 new 생성자 매개변수에 this가 들어가 있는 것을 확인할 수 있는데요, 이것은 생성자를 호출할 때 이 MainFrame의 주소값을 넘겨주어서 각각 연동할 수 있게 설계했습니다. 그리고 버튼과 페이지를 arrayList의 add()메서드로 순서대로 넣어주었습니다! 그리고 저는 5개의 탭을 만들었습니다. bookMain(도서관리), schedule(일정관리), memberRegister(회원가입), CSMain(고객센터), loginForm(로그인,로그아웃)이 그것인데요, 이것은 center패널에 다 붙여버렸습니다 ㅎㅎ; 그럼 어떻게 될까요? 넵 당연히 5개가 서로서로를 덮어서 다 볼 수가 없겠죠.. 그래서 한 화면이 활성화되면 나머지는 보이지 않게 하는 메서드를 만들었습니다..

```
	public void showHide(int n) {
		for (int i = 0; i < pageList.size(); i++) {
			Page page = pageList.get(i);
			if (i == n) {
				page.setVisible(true);
			} else {
				page.setVisible(false);
			}
		}
	}
```

이건데요 들어온 매개변수를 순서와 함께 저장되어있는 페이지리스트에서 추출하고 나머진 setVisible(false) 로 해서 보이지 않게 했습니다!

```
	public void actionPerformed(ActionEvent e) {
		JButton btn = (JButton) e.getSource();

		int index = btnList.indexOf(btn);
		if (loginObj == null) {
			if (index == 0 || index == 1 || index == 3) {
				JOptionPane.showMessageDialog(this, "로그인이 필요한 서비스입니다.");
			} else { // 2,4
				showHide(index);// 어떤 버튼을 누르면 그 인덱스 추출해서 매개변수로 전달
			}
		} else {
			// 로그인한 상태에서 만일 로그아웃 버튼 누르면?
			if (index == 4) {// 로그아웃 대상
				showHide(index);
				loginObj = null;
				JOptionPane.showMessageDialog(this, "로그아웃 되었습니다.");
				bt_login.setText("Login");
			} else {
				showHide(index);
			}
		}
	}
```

이것은 액션리스너입니다. 이제 여기 버튼타입의 btn을 만들고 거기에 우리가 누르는 버튼의 정보를 담아둡니다. 그리고 정수타입 인덱스에 그 리스트의 순서번호를 가져옵니다. 그러면 첫번째 버튼의 페이지는 0번이고 그 다음 1번.. 이렇게 가겠네요 그래서 누를때마다 해당 페이지를 보여주는 것입니다. 그담에 설명은 회원가입과 로그인 페이지를 설명하고 다시 오겠습니다!

![Desktop View](/assets/img/Programming-Language/Java/GUI-Book-App/1.png)

잘 나오네요 ^^ ㅎㅎ 재밌습니다

//MemberRegister

```
//도서관리 페이지
public class MemberRegister extends Page{
	
	JLabel la_id, la_pass,la_name,la_email;
	JTextField t_id,t_pass,t_name,t_email;
	JButton bt_regist;
	
	JPanel content;
	AdminDAO adminDAO;
	
	public MemberRegister(MainFrame mainFrame) {
		super(mainFrame);
		//setBackground(Color.BLUE);
		
		//생성 
		la_id = new JLabel("ID");
		la_pass = new JLabel("Password");
		la_name = new JLabel("Name");
		la_email = new JLabel("Email");
		
		t_id =new JTextField();
		t_pass =new JTextField();
		t_name =new JTextField();
		t_email =new JTextField();
		bt_regist = new JButton("회원가입");
	
		content = new JPanel();
		adminDAO = new AdminDAO();
		
		
		//스타일
		Dimension d = new Dimension(400,40);
		la_id.setPreferredSize(d);
		la_pass.setPreferredSize(d);
		la_name.setPreferredSize(d);
		la_email.setPreferredSize(d);
		
		t_id.setPreferredSize(d);
		t_pass.setPreferredSize(d);
		t_name.setPreferredSize(d);
		t_email.setPreferredSize(d);
		
		content.setPreferredSize(new Dimension(830,300));
		
		//조립
		content.add(la_id);
		content.add(t_id);
		content.add(la_pass);
		content.add(t_pass);
		content.add(la_name);
		content.add(t_name);
		content.add(la_email);
		content.add(t_email);
		content.add(bt_regist);
		add(content);
		
		//리스너 연결
		bt_regist.addActionListener(new ActionListener() {
			
			public void actionPerformed(ActionEvent e) {
				String id = t_id.getText();
				String pass = t_pass.getText();
				String name = t_name.getText();
				String email = t_email.getText();
				
				//낱개로 보내지말고 , 하나의 객체의 인스턴스에 담아서 전달하자!(VO이용해본다)
				Admin admin = new Admin();//empty
				admin.setId(id);
				admin.setName(name);
				admin.setPassword(pass);
				admin.setEmail(email);
				
				int result = adminDAO.insert(admin);
				
				if(result ==1) {
					JOptionPane.showMessageDialog(getMainFrame(), "회원가입 성공");
				}else {
					JOptionPane.showMessageDialog(getMainFrame(), "회원가입 성공");
				}
			}
		});
	}
}
```

회원가입 버튼을 누르면 이페이지가 나오겠죠? 디자인 부분은 생략하고 회원가입 버튼을 누르면 어떻게 되는지 봅시다.

```
		bt_regist.addActionListener(new ActionListener() {
			
			public void actionPerformed(ActionEvent e) {
				String id = t_id.getText();
				String pass = t_pass.getText();
				String name = t_name.getText();
				String email = t_email.getText();
				
				//낱개로 보내지말고 , 하나의 객체의 인스턴스에 담아서 전달하자!(VO이용해본다)
				Admin admin = new Admin();//empty
				admin.setId(id);
				admin.setName(name);
				admin.setPassword(pass);
				admin.setEmail(email);
				
				int result = adminDAO.insert(admin);
				
				if(result ==1) {
					JOptionPane.showMessageDialog(getMainFrame(), "회원가입 성공");
				}else {
					JOptionPane.showMessageDialog(getMainFrame(), "회원가입 성공");
				}
			}
		});
```

우리가 입력한 것들이 id,pass,name,email로 들어갑니다. 그리고 미리 만든 Admin타입의 세터들로 보냅니다. 그럼 이제 admin으로 가볼까요?

//admin

```
package com.koreait.bookapp.model;

//아래의 클래스는 평상시와는 달리, 로직을 작성하기 위함이 아니라 ,오직 DB레코드 한건을 담기 위한
//용도의 객체이다. 이러한 목적으로 정의된 객체를 가리켜, 어플리케이션 분야에서는 VO(Value Objet,DTO(Data Transfer Object)

public class Admin {
	private int admin_id;
	private String id;
	private String name;
	private String password;
	private String email;
	
	public int getAdmin_id() {
		return admin_id;
	}
	public void setAdmin_id(int admin_id) {
		this.admin_id = admin_id;
	}
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
}
```

보면 다른 기능적인 메서드는 없고 멤버변수로서 존재하고 다 게터세터처리가 되어있습니다. 아래의 클래스는 평상시와는 달리, 로직을 작성하기 위함이 아니라 ,오직 DB레코드 한건을 담기 위한 용도의 객체입니다. 이러한 목적으로 정의된 객체를 가리켜, 어플리케이션 분야에서는 VO(Value Objet,DTO(Data Transfer Object)라고 합니닷!

그럼 다시 MemberRegist클래스의 회원가입 액션리스너로 가면 저 Admin클래스로 들어간 정보들을 adminDAO에 있는 메서드 insert()의 매개변수로 정보가 들어갑니다. 그리고 그것을 정수형 result에 넣어서 값을 반환해주겠죠 우리는 한줄씩 만들거라 1이 반환되면 회원가입이 성공한겁니다!그럼 이제 adminDAO를 살펴보겠습니다!

//AdminDAO

```
package com.koreait.bookapp.model;

public class AdminDAO {
	String url = "jdbc:oracle:thin:@localhost:1521:XE";
	String user = "batman";
	String pass = "1234";
	// Admin 테이블에 레코드 넣기!

	public int insert(Admin admin) {
		Connection con = null;
		PreparedStatement pstmt = null;
		int result = 0;
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");// 드라이버 로드 시도
			con = DriverManager.getConnection(url, user, pass);// 접속시도
			if (con != null) {
				System.out.println("접속성공");

				String sql = "insert into admin(admin_id,id,name,pass,email)";
				sql += " values(seq_admin.nextval,?,?,?,?)";

				pstmt = con.prepareStatement(sql);// 쿼리 객체 생성

				// 바인드 변수에 값 할당하기 바인드 변수는 1부터 시작
				pstmt.setString(1, admin.getId());
				pstmt.setString(2, admin.getName());
				pstmt.setString(3, admin.getPassword());
				pstmt.setString(4, admin.getEmail());

				result = pstmt.executeUpdate();// 이 쿼리문에 의해 영향을 받은 레코드 수를 반환

			} else {
				System.out.println("접속실패");
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if (pstmt != null) {
				try {
					pstmt.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (con != null) {
				try {
					con.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return result;
	}

	// 존재하는 회원인지 여부 처리 메서드(로그인 처리)
	public Admin select(Admin admin) {
		Connection con = null;
		PreparedStatement pstmt=null;
		ResultSet rs =null;
		Admin vo = null;//반환하기 위해 변수 위로 올림
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");//드라이버 로드
			con = DriverManager.getConnection(url, user, pass);

			if (con != null) {
				System.out.println("접속성공");
				String sql = "select * from admin where id=? and pass =?";
				pstmt = con.prepareStatement(sql);//쿼리수행 객체 얻기
				pstmt.setString(1, admin.getId() ); //VO에 들어있는 id값
				pstmt.setString(2, admin.getPassword());//vo에 들어있는 password값
				
				rs =pstmt.executeQuery();//select문 수행 후 그 결과를 ResultSet으로 받음
				
				if(rs.next()) {
					System.out.println("레코드가 존재하므로 ,로그인 성공");
					//로그인에 성공하면, 어플리케이션은 해당 회원의 정보를 언제든 출력하거나, 수정할 수 있게 해줘야 한다..
					//따라서 이 메서드 반환형으로 회원정보를 반환해주자!
					vo = new Admin();//empty 인스턴스 생성
					vo.setAdmin_id(rs.getInt("admin_id"));//pk담았음
					vo.setId(rs.getString("id"));
					vo.setPassword(rs.getString("pass"));
					vo.setName(rs.getString("name"));
					vo.setEmail(rs.getString("email"));
				}else {
					System.out.println("레코드 존재하지 않음, 실패");
				}
				
			} else {
				System.out.println("접속실패");
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if (rs != null) {
				try {
					rs.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (pstmt != null) {
				try {
					pstmt.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (con != null) {
				try {
					con.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return vo;
	}
}
```

DAO란? Data Access Object : 오직 Create (insert),Read (select),Update, Delete 작업만을 수행하는 모델 객체를 말합니다. 따라서 우리가 Admin타입을 보내면 여기서 그 정보를 바탕으로 여러가지 쿼리문을 실행하는 로직공간입니다! 여기엔 insert(회원가입)메서드와 select(로그인)메서드가 있는데 일단 지금은 insert()메서드만 볼게요!

```
	public int insert(Admin admin) {
		Connection con = null;
		PreparedStatement pstmt = null;
		int result = 0;
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");// 드라이버 로드 시도
			con = DriverManager.getConnection(url, user, pass);// 접속시도
			if (con != null) {
				System.out.println("접속성공");

				String sql = "insert into admin(admin_id,id,name,pass,email)";
				sql += " values(seq_admin.nextval,?,?,?,?)";

				pstmt = con.prepareStatement(sql);// 쿼리 객체 생성

				// 바인드 변수에 값 할당하기 바인드 변수는 1부터 시작
				pstmt.setString(1, admin.getId());
				pstmt.setString(2, admin.getName());
				pstmt.setString(3, admin.getPassword());
				pstmt.setString(4, admin.getEmail());

				result = pstmt.executeUpdate();// 이 쿼리문에 의해 영향을 받은 레코드 수를 반환

			} else {
				System.out.println("접속실패");
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if (pstmt != null) {
				try {
					pstmt.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (con != null) {
				try {
					con.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return result;
	}
```

오라클 드라이버를 로드하고 멤버변수로 설정한 url 과 user와 pass로 오라클서버에 접속을 해주고 그 정보를 con에 넣었습니다. 그리고 if문 조건문으로 접속이 성공했다면 미리 준비한 insert명령어를 품은 sql문자열에 쿼리객체를 또 생성했습니다. 그런데 오잉? 중간에 ? 라는 물음표 문자가 있네요? 이것은 바인드 문자인데요 나중에 set관련된 메서드로 후에 그 값을 수정할 수 있습니다. 그래서 밑에 보시면

pstmt.setString(1, admin.getId()); 라는 문장이있죠? 이것은 1, 즉 첫번째 바인드문자 (=?)에 문자열을 넣어주는 것입니다. 넣어준 값은? 이미 가져온 우리가 회원가입할 때 적은 정보들이 적혀있는 admin 에 있는 id와 name ,password, email이 들어있죠 이것을 넣습니다. 그럼 저 sql문에는 우리가 작성했던 값들이 들어가서 저 쿼리문이 실행되겠죠? 그래서 result = pstmt.executeUpdate(); 문장으로 쿼리문을 실행하고 실행해서 만들어진 레코드 수를 result에 넣습니다. 우리는 한줄만 만들었으니 1이 들어갔겠죠? 그리고 그 값을 리턴해줍니다! 한번 볼까요?

![Desktop View](/assets/img/Programming-Language/Java/GUI-Book-App/2.png)

![Desktop View](/assets/img/Programming-Language/Java/GUI-Book-App/3.png)

저렇게 입력하고 오라클에서 보니까 잘 정보가 입력되었습니다! 3번째 antman은 ㅋㅋ 비번이랑 이름이랑 헷갈려서 이상하게 들어갔네요.. 멍청이 ㅜ

이제 회원가입도 했으니 로그인을 해봅시다.

//LoginForm

```
package com.koreait.bookapp.view;

public class LoginForm extends Page{
	JTextField t_id;
	JPasswordField t_pass;
	JPanel content;
	JButton bt_login;
	AdminDAO adminDAO;
	
	public LoginForm(MainFrame mainFrame) {
		super(mainFrame);
		
		//생성
		t_id= new JTextField();
		t_pass = new JPasswordField();
		content = new JPanel();
		bt_login = new JButton("Login");
		adminDAO = new AdminDAO();
		
		//스타일
		Dimension d = new Dimension(480,40);
		t_id.setPreferredSize(d);
		t_pass.setPreferredSize(d);
		
		content.setBackground(Color.YELLOW);
		content.setPreferredSize(new Dimension(500,230));
		
		//조립
		content.add(t_id);
		content.add(t_pass);
		content.add(bt_login);
		
		add(content);
	
		bt_login.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				//아이디와 패스워드를 VO담아서 전달!
				String id = t_id.getText();//암시적 생성법
				String pass = new String (t_pass.getPassword()); //apple 'a' 'p' ....'e'
			
				//VO를 생성하여 아이디와 패스워드를 심어서 보내자!!
				Admin admin = new Admin();//empty
				admin.setId(id);
				admin.setPassword(pass);
				Admin result =adminDAO.select(admin);
				if(result ==null) {
					JOptionPane.showMessageDialog(getMainFrame(), "로그인 정보가 올바르지 않습니다.");
				}else {
					JOptionPane.showMessageDialog(getMainFrame(),result.getName() +"님 반갑습니다!");
					//기존 로그인 버튼의 제목을 Logout으로 전환
					getMainFrame().bt_login.setText("Logout");
					getMainFrame().loginObj =result; //로그인 결과 객체를 프레임 대입
				}
			}
		});
	}
}
```

일단 여기서의 로직의 큰 틀은 텍스트 필드에 입력한 id와 pass가 오라클 테이블에 있는 어떤 레코드와 일치하는지를 select문으로 조회하고 조회가 된다면 일치한다는 것이겠죠? 이런식으로 진행하겠습니다! 그전에 디자인 부분에서 눈여결 볼 부분이 있습니다.

```
t_pass = new JPasswordField();
```

이것인데요 보통 어떤 것을 입력할 때는 텍스트필드를 쓰지만 여기에는 JPasswordField를 썼습니다. 다른 점이라면 여기에 입력할때는 입력값이 보이지 않고 특수문자로 표현됩니다!

이제 액션리스너를 중점으로 보겠습니다!

```
		bt_login.addActionListener(new ActionListener() {
			@Override
			public void actionPerformed(ActionEvent e) {
				//아이디와 패스워드를 VO담아서 전달!
				String id = t_id.getText();//암시적 생성법
				String pass = new String (t_pass.getPassword()); //apple 'a' 'p' ....'e'
			
				//VO를 생성하여 아이디와 패스워드를 심어서 보내자!!
				Admin admin = new Admin();//empty
				admin.setId(id);
				admin.setPassword(pass);
				Admin result =adminDAO.select(admin);
				if(result ==null) {
					JOptionPane.showMessageDialog(getMainFrame(), "로그인 정보가 올바르지 않습니다.");
				}else {
					JOptionPane.showMessageDialog(getMainFrame(),result.getName() +"님 반갑습니다!");
					//기존 로그인 버튼의 제목을 Logout으로 전환
					getMainFrame().bt_login.setText("Logout");
					getMainFrame().loginObj =result; //로그인 결과 객체를 프레임 대입
				}
			}
		});
```

입력했던 값은 getText()로 가져와서 id와 pass에 넣었습니다. 다른점은 pass는 패스워드필드라서 getText()가 없습니다. 그래서 getPassword()로 가져온 char형을 래퍼클래스로 String, 즉 문자열로 변환해서 pass에 넣었습니다 그리고 아까 회원가입처럼 일시적으로 쓸 VO Admin타입의 객체 admin을 만들어서 여기에 작성한 id와 pass를 넣었습니다. 그리고 이 값을 AdminDAO에 있는 select()메서드에 넣는군요.. 그럼 아까 insert()메서드만 봤는데 이제 select()메서드를 보겠습니다!

select()

```
	public Admin select(Admin admin) {
		Connection con = null;
		PreparedStatement pstmt=null;
		ResultSet rs =null;
		Admin vo = null;//반환하기 위해 변수 위로 올림
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");//드라이버 로드
			con = DriverManager.getConnection(url, user, pass);

			if (con != null) {
				System.out.println("접속성공");
				String sql = "select * from admin where id=? and pass =?";
				pstmt = con.prepareStatement(sql);//쿼리수행 객체 얻기
				pstmt.setString(1, admin.getId() ); //VO에 들어있는 id값
				pstmt.setString(2, admin.getPassword());//vo에 들어있는 password값
				
				rs =pstmt.executeQuery();//select문 수행 후 그 결과를 ResultSet으로 받음
				
				if(rs.next()) {
					System.out.println("레코드가 존재하므로 ,로그인 성공");
					//로그인에 성공하면, 어플리케이션은 해당 회원의 정보를 언제든 출력하거나, 수정할 수 있게 해줘야 한다..
					//따라서 이 메서드 반환형으로 회원정보를 반환해주자!
					vo = new Admin();//empty 인스턴스 생성
					vo.setAdmin_id(rs.getInt("admin_id"));//pk담았음
					vo.setId(rs.getString("id"));
					vo.setPassword(rs.getString("pass"));
					vo.setName(rs.getString("name"));
					vo.setEmail(rs.getString("email"));
				}else {
					System.out.println("레코드 존재하지 않음, 실패");
				}
				
			} else {
				System.out.println("접속실패");
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			if (rs != null) {
				try {
					rs.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (pstmt != null) {
				try {
					pstmt.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			if (con != null) {
				try {
					con.close();
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
		}
		return vo;
	}
```

좀 길죠? 셀렉문이라 ResultSet도 사용해서 이것도 나중에 close()메서드로 해제해주어야해서 좀 길어졌습니다. 먼저 이것도 드라이버를 로드하고 접속까지 했습니다. 접속이 성공했다면 미리 준비한 select수행문을 sql이란 문자열에 넣어주고 바인드 변수도 넣었습니다. 그렇다면 아까 우리가 입력한 id와 pass가 Admin타입으로 admin객체에 정보가 들어가고 이 객체가 여기까지 흘러왔습니다. 그리고 그 가져온 값의 id와 pass를 바인드 변수에 넣고 실행을 했습니다. 그렇다면 어떻게 될까요? select \* from admin where id=? and pass =? 여기에 아이디와 패스워드를 넣고 오라클에서 조회를 한다면 만약 일치한다면 레코드 하나가 조회되겠죠? 그러면 입력한 아이디와 비밀번호가 데이터베이스에 있는 어떤 레코드와 일치한다는 말입니다. 그렇다면 로그인에 성공한 것이지요. 그래서 if(rs.next())문으로 조회된 테이블에 next()가 false가 아니라 true라면, 즉 레코드가 조회되었다면 일치한 아이디와 비밀번호가 있을거고 다시 말하지만 로그인에 성공한거죠? 성공하였으니 보상을 주도록 할게요. 같은 Admin타입이지만 다른 객체인 vo를 만들었습니다. 여기에 조회된 정보를 전부 넣어줍니다. 그리고 이 전체 select()메서드의 반환값으로서 정보를 vo로 반환하죠. 그리고 이것을 LoginForm클래스에 있는 loginObj에 넣어줍니다. 여기엔 Admin타입의 객체 vo가 있겠죠? 나중에 이것으로 로그인의 여부를 판단합니다.

로그인 해볼까요? 아까만든 spiderman /1234로 로그인 해봅시다

![Desktop View](/assets/img/Programming-Language/Java/GUI-Book-App/4.png)

값을 입력하고 버튼을 누르니 로그인이 잘 됐다고 나오네요 ^^ 뿌듯합니다 이제 다시 MainFrame에 있는 액션리스너를 다시 볼까요?

```
	public void actionPerformed(ActionEvent e) {
		JButton btn = (JButton) e.getSource();

		int index = btnList.indexOf(btn);
		if (loginObj == null) {
			if (index == 0 || index == 1 || index == 3) {
				JOptionPane.showMessageDialog(this, "로그인이 필요한 서비스입니다.");
			} else { // 2,4
				showHide(index);// 어떤 버튼을 누르면 그 인덱스 추출해서 매개변수로 전달
			}
		} else {
			// 로그인한 상태에서 만일 로그아웃 버튼 누르면?
			if (index == 4) {// 로그아웃 대상
				showHide(index);
				loginObj = null;
				JOptionPane.showMessageDialog(this, "로그아웃 되었습니다.");
				bt_login.setText("Login");
			} else {
				showHide(index);
			}
		}
	}
```

보시면 if문 조건안에 loginObj ==null이 보이시나요? 이것은 아직 로그인을 안 한 상태입니다. 그럴때 인덱스 0,1,3은 못들어가게 막았죠? 아닐 때는 showHide()메서드로 회원가입과 로그인페이지만 볼수있습니다. 만약 로그인을 했다면 인덱스 4, 즉 원래 로그인 버튼을 로그아웃으로 바꾸고 다시 그것을 눌렀을 때 loginObj에 null값을 주면서 다시 로그아웃을 시키는 것이죠!

이번시간에는 여기까지하고 다음에 또 업데이트 하겠습니다!!