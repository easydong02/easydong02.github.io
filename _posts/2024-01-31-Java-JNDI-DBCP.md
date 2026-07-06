---
title: "[Java] JNDI와 DBCP"
date: 2024-01-31 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, jndi, dbcp]
render_with_liquid: false
---

## 📌 들어가며

데이터베이스 연결을 매번 새로 만드는 것은 비용이 큰 작업이다. 또 접속 정보를 코드에 하드코딩하면 환경이 바뀔 때마다 수정해야 한다. 이 두 문제를 각각 **DBCP(연결 재사용)**와 **JNDI(설정 외부화)**가 해결한다. 두 기술을 함께 쓰면 효율과 유지보수성을 동시에 잡을 수 있다.

| 기술 | 해결하는 문제 |
|------|------|
| **JNDI** | 접속 정보를 코드 밖(중앙)에서 관리 |
| **DBCP** | 연결을 미리 만들어 풀에 두고 재사용 |

---

## 1. JNDI (Java Naming and Directory Interface)

**JNDI**는 자바 애플리케이션에서 네이밍/디렉터리 서비스에 접근하는 API다. 주로 분산 환경에서 객체를 찾고, DB 연결 정보 같은 환경 설정을 **중앙에서 관리**한다.

**사용 이유:**

- **중앙화된 환경 설정**: 접속 정보를 코드에 하드코딩하지 않고 중앙에서 관리
- **코드 수정 없이 변경**: 설정이 바뀌어도 애플리케이션 코드는 그대로

```java
// JNDI로부터 데이터베이스 연결 가져오기
Context initialContext = new InitialContext();
DataSource dataSource =
    (DataSource) initialContext.lookup("java:comp/env/jdbc/myDataSource");

try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
} catch (SQLException e) {
    e.printStackTrace();
}
```

`"java:comp/env/jdbc/myDataSource"`는 JNDI 네이밍 규칙에 따른 **이름**이다. 이 이름으로 등록된 데이터 소스를 찾아 반환한다.

---

## 2. DBCP (Database Connection Pooling)

**DBCP**는 데이터베이스 연결을 미리 생성해 **풀(pool)**에 저장했다가, 필요할 때 꺼내 쓰고 다시 반납하는 기술이다.

```
[DBCP 없이]  요청마다  연결 생성 → 사용 → 종료   (매번 비싼 비용)

[DBCP]       풀에서 연결 대여 → 사용 → 반납        (재사용)
             ┌───────── Connection Pool ─────────┐
             │  [conn] [conn] [conn] [conn] ...  │
             └───────────────────────────────────┘
```

**사용 이유:**

| 이유 | 설명 |
|------|------|
| **성능 향상** | 연결 생성은 비용이 큰 작업 → 미리 만든 연결 재활용 |
| **커넥션 누수 방지** | 반환되지 않은 연결을 관리하는 메커니즘 제공 |

```java
// DBCP 설정 (Apache Commons DBCP)
BasicDataSource dataSource = new BasicDataSource();
dataSource.setDriverClassName("com.mysql.jdbc.Driver");
dataSource.setUrl("jdbc:mysql://localhost:3306/mydatabase");
dataSource.setUsername("username");
dataSource.setPassword("password");
dataSource.setInitialSize(5);   // 초기 연결 수
dataSource.setMaxTotal(10);     // 최대 연결 수

try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
} catch (SQLException e) {
    e.printStackTrace();
}
```

`BasicDataSource`는 Apache Commons DBCP가 제공하는 클래스로, 연결 풀을 설정·관리한다.

---

## 3. JNDI + DBCP 함께 쓰기

JNDI로 **연결 정보를 중앙 관리**하고, 그 뒤에서 DBCP가 **연결 풀을 관리**하도록 결합한다.

```java
// JNDI로부터 (DBCP로 관리되는) DataSource 조회
Context initialContext = new InitialContext();
DataSource dataSource =
    (DataSource) initialContext.lookup("java:comp/env/jdbc/myDataSource");

try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
} catch (SQLException e) {
    e.printStackTrace();
}
```

애플리케이션 코드는 **이름으로 lookup만** 하면 되고, 그 뒤의 풀 관리와 설정은 컨테이너/DBCP가 담당한다.

---

## 📌 성능 최적화 팁

| 설정 | 옵션 | 효과 |
|------|------|------|
| **풀 크기 조절** | `setInitialSize`, `setMaxTotal` | 초기/최대 연결 수를 자원에 맞게 조절 |
| **유휴 연결 검사** | `setTestWhileIdle`, `setTimeBetweenEvictionRunsMillis` | 사용 안 하는 연결을 주기적으로 정리 |

---

## 📝 정리

```
JNDI + DBCP
├─ JNDI  이름으로 리소스 접근, 접속 정보 중앙 관리
├─ DBCP  연결 풀에서 대여·반납 → 재사용으로 성능↑, 누수 방지
└─ 결합  lookup으로 조회 + 뒤에서 풀 관리 (설정 외부화 + 성능)
```

| 개념 | 한 줄 정의 |
|------|------|
| **JNDI** | 이름으로 리소스에 접근하는 API (설정 외부화) |
| **DBCP** | 연결을 풀에 두고 재사용하는 기술 |
| **BasicDataSource** | Commons DBCP의 연결 풀 구현체 |

JNDI는 "설정을 코드 밖으로", DBCP는 "연결을 재사용으로". 이 둘을 결합하면 유지보수성과 성능을 함께 확보하는, 실무의 표준적인 DB 연결 관리 방식이 완성된다.
