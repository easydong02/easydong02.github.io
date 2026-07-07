---
title: "[Mybatis] 마이바티스 원리와 사용법"
date: 2023-12-21 00:00:00 +0900
categories: [Backend, Mybatis]
tags: [mybatis]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 **MyBatis**의 원리와 사용법을 정리한다.

![Desktop View](/assets/img/Backend/Mybatis/Mybatis/1.jpg)

> **MyBatis란?** 스프링에서 **데이터베이스 연동을 도와주는 프레임워크**. JDBC의 중복 코드를 제거하고, **SQL을 XML로 분리**해 관리한다. 편리하지만 단점이 있어 JPA와 함께 쓰면 좋다.

---

## 1. 기존 JDBC의 문제

기존 방식은 **하나의 DAO에 연결·SQL·해제**가 모두 뒤섞여 있었다.

```java
@Override
public int writeArticle(BoardDto boardDto) throws SQLException {
    Connection conn = null;
    PreparedStatement pstmt = null;
    try {
        conn = dataSource.getConnection();               // 연결
        StringBuilder sql = new StringBuilder();
        sql.append("insert into board (...) values (?, ?, ?, 0, now())");  // SQL
        pstmt = conn.prepareStatement(sql.toString());
        pstmt.setString(1, boardDto.getUserId());        // 바인딩
        // ...
        return pstmt.executeUpdate();
    } finally {
        dbUtil.close(pstmt, conn);                       // 해제
    }
}
```

> ⚠️ 소스코드 안에 SQL이 들어가고, 드라이버 등록부터 연결 해제까지 **반복 코드**가 매번 필요했다. MyBatis는 이를 **인터페이스 + XML**로 분리한다.

---

## 2. MyBatis 구성 요소

| 구성 요소 | 역할 |
|------|------|
| **Configuration file** (XML) | 연결 대상·매핑 경로 등 설정 (스프링 통합 시 생략 가능) |
| **SqlSessionFactory** | SqlSession 생성 (스프링이 대신 처리) |
| **SqlSession** | SQL 실행·트랜잭션 제어 API (가장 중요) |
| **Mapper interface** | 매핑 XML의 SQL을 호출하는 인터페이스 (**구현 자동 생성**) |
| **Mapping file** (XML) | SQL과 O/R 매핑 설정 |

**Mapper 인터페이스** — 메소드 선언만:

```java
@Mapper
public interface BoardMapper {
    void write(BoardDto board);
    List<BoardDto> list();
    BoardDto getBoard(int articleNo);
}
```

**Mapping XML** — SQL을 여기에 작성:

```xml
<mapper namespace="com.ssafy.board.model.mapper.BoardMapper">
    <select id="list" resultMap="article">
        select b.article_no, b.user_id, m.user_name, b.subject ...
        from board b, members m where b.user_id = m.user_id
    </select>

    <insert id="write" parameterType="boardDto">
        insert into board(user_id, subject, content, hit, register_time)
        values(#{userId}, #{subject}, #{content}, 0, now())
    </insert>
</mapper>
```

> 💡 **인터페이스의 메소드 이름(`id`)과 XML의 SQL이 매핑**된다. `#{userId}`처럼 파라미터를 바인딩하고, `resultMap`으로 컬럼↔프로퍼티를 매핑한다. 구현체는 MyBatis가 자동 생성한다.

---

## 3. DB 접근 순서

![Desktop View](/assets/img/Backend/Mybatis/Mybatis/3.png)

```
[시작 시 1회]
① 앱 → SqlSessionFactory 빌드 요청
② 구성 파일 읽기
③ SqlSessionFactory 생성

[요청마다]
④~⑥ SqlSessionFactory → SqlSession 생성
⑦~⑧ 앱 → Mapper 구현체 → 메소드 호출
⑨~⑩ SqlSession이 XML의 SQL을 가져와 실행
```

**MyBatis-Spring 통합** 시에는 `SqlSessionFactoryBean`, `MapperFactoryBean`, `SqlSessionTemplate`이 **스레드 안전한 매퍼를 만들어 DI**로 주입한다.

![Desktop View](/assets/img/Backend/Mybatis/Mybatis/4.png)

---

## 4. MyBatis vs JPA

| MyBatis 단점 | JPA 장점 |
|------|------|
| 스키마 변경 시 SQL 직접 수정 | 쿼리 작성 불필요, 코드량↓ |
| 반복 쿼리 작업 | 가독성 좋음 |
| DB 종속 쿼리 발생 | 간편한 수정 |
| DB 변경 시 로직 수정 | 캐시로 성능↑ |

> 💡 **결론**: **CRUD 같은 단순 코드는 JPA, 복잡한 쿼리는 MyBatis**로 사용하면 좋다. 서로의 장단점을 보완한다.

---

## 📝 정리

```
MyBatis
├─ 개념   SQL을 XML로 분리한 DB 연동 프레임워크
├─ 구성   Mapper 인터페이스 + Mapping XML (id로 매핑)
├─ 순서   SqlSessionFactory → SqlSession → Mapper → SQL 실행
└─ 조합   단순 CRUD는 JPA, 복잡 쿼리는 MyBatis
```

| 개념 | 한 줄 정의 |
|------|------|
| **MyBatis** | SQL을 XML로 분리하는 프레임워크 |
| **Mapper** | 인터페이스 ↔ XML SQL 매핑 |
| **SqlSession** | SQL 실행·트랜잭션 API |

MyBatis의 핵심은 **"SQL을 코드에서 XML로 분리"**하는 것이다. JDBC의 반복을 없애주지만 쿼리를 직접 관리해야 하므로, JPA와 역할을 나눠 쓰는 것이 실무의 좋은 전략이다.
