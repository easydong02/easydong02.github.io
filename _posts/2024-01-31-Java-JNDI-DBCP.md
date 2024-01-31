---
title: "[Java] JNDI와 DBCP"
date: 2024-01-31 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, jndi, dbcp]
render_with_liquid: false
---

# **JNDI와 DBCP: Java에서의 데이터베이스 연결 관리**

## ✅**소개**

---

Java에서 데이터베이스와의 연결을 효과적으로 관리하기 위해서는 JNDI(Java Naming and Directory Interface)와 DBCP(Database Connection Pooling)을 활용하는 것이 일반적입니다. 이 두 기술을 조합하면 데이터베이스 연결의 효율성을 높이고, 성능을 향상시킬 수 있습니다. 이번 글에서는 JNDI와 DBCP의 개념과 함께 예시 코드를 통해 실제로 어떻게 사용하는지에 대해 살펴보겠습니다.

## ✅**JNDI(Java Naming and Directory Interface)**

---

### **JNDI란?**

JNDI는 Java 애플리케이션에서 네이밍 서비스에 접근하고 디렉터리 서비스를 사용할 수 있도록 하는 Java API입니다. 주로 분산 환경에서 객체를 찾고 검색하는 데 사용되며, 데이터베이스 연결 정보와 같은 환경 설정을 중앙에서 관리할 수 있도록 지원합니다.

### **JNDI를 사용하는 이유**

1. **중앙화된 환경 설정:** 데이터베이스 연결 정보나 기타 환경 설정을 애플리케이션 코드에 하드코딩하지 않고 중앙에서 효과적으로 관리할 수 있습니다.
2. **코드의 수정 없이 변경 가능:** 설정이 변경되더라도 애플리케이션 코드를 수정할 필요 없이 외부에서 설정을 변경할 수 있습니다.

### **JNDI 사용 예시**

```java
// JNDI로부터 데이터베이스 연결 가져오기
Context initialContext = new InitialContext();
DataSource dataSource = (DataSource) initialContext.lookup("java:comp/env/jdbc/myDataSource");

// 데이터베이스 연결 사용 예시
try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
    // ...
} catch (SQLException e) {
    e.printStackTrace();
}

```

위 예시에서 "java:comp/env/jdbc/myDataSource"는 JNDI 네이밍 규칙에 따른 데이터베이스 연결 정보의 이름입니다. 이 이름을 통해 JNDI는 등록된 데이터 소스를 찾아서 반환합니다.

## ✅**DBCP(Database Connection Pooling)**

---

### **DBCP란?**

DBCP는 데이터베이스 연결을 미리 생성해두고 풀(pool)에 저장해두었다가 필요할 때 풀에서 꺼내어 사용하는 기술입니다. 이를 통해 데이터베이스 연결을 매번 새로 만들지 않고 재사용함으로써 성능을 향상시킬 수 있습니다.

### **DBCP를 사용하는 이유**

1. **성능 향상:** 데이터베이스 연결 생성은 비용이 큰 작업 중 하나입니다. DBCP를 사용하면 미리 생성된 연결을 재활용함으로써 성능을 향상시킬 수 있습니다.
2. **커넥션 누수 방지:** 애플리케이션이 데이터베이스 연결을 사용한 후에 반드시 반환하지 않는 경우, 해당 연결은 누수되어 풀에서 제대로 관리되지 못합니다. DBCP는 이를 방지하기 위한 메커니즘을 제공합니다.

### **DBCP 사용 예시**

```java
// DBCP 설정
BasicDataSource dataSource = new BasicDataSource();
dataSource.setDriverClassName("com.mysql.jdbc.Driver");
dataSource.setUrl("jdbc:mysql://localhost:3306/mydatabase");
dataSource.setUsername("username");
dataSource.setPassword("password");
dataSource.setInitialSize(5); // 초기 연결 수
dataSource.setMaxTotal(10); // 최대 연결 수

// 데이터베이스 연결 사용 예시
try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
    // ...
} catch (SQLException e) {
    e.printStackTrace();
}

```

위 예시에서 **`BasicDataSource`**는 Apache Commons DBCP 라이브러리에서 제공하는 클래스로, 데이터베이스 연결 풀을 설정하고 관리합니다.

## ✅**JNDI와 DBCP의 함께 사용**

---

JNDI와 DBCP를 함께 사용하면 JNDI를 통해 데이터베이스 연결 정보를 중앙에서 관리하고, DBCP를 사용하여 효과적으로 데이터베이스 연결을 관리할 수 있습니다.

```java
// JNDI로부터 데이터베이스 연결 가져오기
Context initialContext = new InitialContext();
DataSource dataSource = (DataSource) initialContext.lookup("java:comp/env/jdbc/myDataSource");

// DBCP를 사용하여 데이터베이스 연결 관리
try (Connection connection = dataSource.getConnection()) {
    // 데이터베이스 작업 수행
    // ...
} catch (SQLException e) {
    e.printStackTrace();
}

```

이렇게 함으로써 중앙에서 데이터베이스 연결 정보를 유연하게 관리하고, DBCP를 통해 성능 향상 및 커넥션 누수 방지 효과를 얻을 수 있습니다.

## 📌**활용 팁: Connection Pool과 성능 최적화**

---

JNDI와 DBCP를 사용하여 Java에서 데이터베이스 연결을 효과적으로 관리하는 방법을 살펴보았습니다. 이제 몇 가지 활용 팁을 통해 더 나은 성능과 안정성을 어떻게 확보할 수 있는지 알아보겠습니다.

### **1. Connection Pool 크기 조절**

DBCP에서는 **`setInitialSize`** 및 **`setMaxTotal`**과 같은 설정을 통해 Connection Pool의 크기를 조절할 수 있습니다. 초기 연결 수는 애플리케이션 기동 시 미리 생성할 연결의 수이며, 최대 연결 수는 풀이 제공할 수 있는 최대 연결 수를 나타냅니다. 적절한 크기로 조절하여 시스템의 자원을 효율적으로 활용하세요.

### **2. 유휴 연결 검사 설정**

DBCP는 **`setTestWhileIdle`** 및 **`setTimeBetweenEvictionRunsMillis`**와 같은 옵션을 제공하여 유휴 연결을 주기적으로 검사할 수 있습니다. 유휴 연결 검사를 통해 사용하지 않는 연결을 풀에서 제거하여 성능을 향상시킬 수 있습니다.