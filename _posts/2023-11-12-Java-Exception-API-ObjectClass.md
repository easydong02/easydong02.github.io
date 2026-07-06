---
title: "[Java] 예외처리 응용, API, Object클래스"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **예외 처리 응용 문제**를 풀고, **API(내부·외부)**를 다루는 법, 그리고 모든 클래스의 부모인 **Object 클래스**(`toString`, `equals`)를 정리한다.

---

## 1. 예외 처리 응용

**문제**: 무한 반복 안에서 정수 5개만 입력받아 5칸 배열에 넣기. **`try~catch`만** 사용.

```java
public class ExceptionTask {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int[] arData = new int[5];
        int tmp = 0;
        String temp = null;
        while (true) {
            System.out.println(++tmp + "번째 정수 입력[나가기 : q]: ");
            try {
                temp = sc.next();
                if (temp.equals("q")) break;
                arData[tmp - 1] = Integer.parseInt(temp);
                System.out.println("추가 성공");
            } catch (NumberFormatException e) {        // 정수가 아닌 입력
                System.out.println("정수만 입력해주세요.");
                tmp--;
            } catch (ArrayIndexOutOfBoundsException e) { // 6번째 입력 시도
                System.out.println("더 이상 정수를 입력할 수 없습니다.");
                tmp--;
            } catch (Exception e) {                     // 그 외
                System.out.println(e);
            }
        }
    }
}
```

두 가지 예외를 각각 처리한 것이 포인트다.

| 예외 | 발생 상황 | 처리 |
|------|------|------|
| `NumberFormatException` | 정수가 아닌 문자 입력 | 안내 + `tmp--`(칸 건너뜀 방지) |
| `ArrayIndexOutOfBoundsException` | 6번째 정수 입력 시도 | 안내 + `tmp--` |

> 💡 `sc.nextInt()` 대신 `Integer.parseInt(sc.next())`를 쓴 이유: 정수가 아닌 입력을 **일단 받아들인 뒤** `NumberFormatException`으로 잡기 위해서다. `nextInt()`는 예외 후 무한 루프에 빠질 수 있다.

---

## 2. API (Application Programming Interface)

> **API란?** 선배 개발자들이 미리 만들어둔 소스코드의 집합.

| 종류 | 설명 |
|------|------|
| **내부 API** | JDK 설치 시 제공되는 기본 API (`docs.oracle.com/javase`) |
| **외부 API** | 업체·기업이 만든 패키지. 보통 **JAR 파일**로 배포 |

### 외부 JAR을 Build Path에 추가

```
프로젝트 우클릭 → Build Path → Configure Build Path
→ Libraries 탭 → Add External JARs → .jar 선택
→ Order and Exports 탭 → Select All → Apply and Close
```

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/1.png)
![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/5.png)
![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/8.png)

추가한 JAR은 `Referenced Libraries`에 나타나며, import 후 사용할 수 있다.

```java
import api.Calc;

public class ApiTest {
    public static void main(String[] args) {
        Calc c = new Calc();
        c.add(10, 9);   // 자동완성으로 메소드 시그니처·설명 확인 가능
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/11.png)
![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/12.png)

### 내가 만든 클래스를 JAR로 배포 (JavaDoc 주석)

```java
/**
 * @author 신동희
 * <br>계산기<br>
 * @since JDK8
 */
public class Calc {
    /**
     * @param num1 : First integer to divide
     * @param num2 : Last integer to divide
     * @return integer value
     * @throws ArithmeticException
     */
    public int div(int num1, int num2) {
        return num1 / num2;
    }
}
```

> 💡 `/** ... */`는 **JavaDoc(어노테이션) 주석**이다. 외부 배포 시 받는 사람이 API 정보를 알 수 있게 해준다. `@author`(만든 사람), `@param`(매개변수 설명), `@return`(반환), `@throws`(예외), `@since`(버전)를 적는다.

`클래스 우클릭 → Export → JAR file`로 JAR을 생성한다.

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/14.png)
![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/16.png)

---

## 3. Object 클래스 — 최상위 부모

모든 클래스의 부모인 `Object`에는 `toString()`, `equals()`가 있다.

### toString()

객체를 출력할 때 항상 `toString()`이 **생략되어 호출**된다. 결과가 마음에 안 들면 재정의한다.

```java
public class Student {
    int number; String name; int age;
    // 생성자 생략

    @Override
    public String toString() {
        return "번호: " + number + "\n이름 : " + name + "\n나이 : " + age + "살";
    }
}
```

```java
Student 신동희 = new Student(1, "신동희", 20);
System.out.println(신동희);   // 주소값 대신 재정의한 문자열 출력
```

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/17.png)

### equals() — `==`는 주소값 비교

> `==`는 사실 **주소값 비교**다. `String`은 `equals`를 **값 비교로 재정의**했다.

```java
String data1 = "ABC";           // 값 중심 (constant pool)
String data2 = "ABC";
String data3 = new String("ABC");   // 주소값 중심 (new)
String data4 = new String("ABC");

System.out.println(data1 == data2);        // true  (같은 pool 주소)
System.out.println(data1.equals(data2));   // true
System.out.println(data3 == data4);        // false (다른 주소)
System.out.println(data3.equals(data4));   // true  (값 비교!)
```

![Desktop View](/assets/img/Programming-Language/Java/Exception-API-ObjectClass/18.png)

`data3 == data4`가 false인데 `equals`는 왜 true일까? **String의 `equals`는 값을 한 글자씩 비교**하도록 재정의됐기 때문이다.

```java
// String.equals() (요약)
public boolean equals(Object anObject) {
    if (this == anObject) return true;             // 주소 같으면 true
    if (anObject instanceof String) {              // 타입 비교
        String anotherString = (String) anObject;
        if (value.length == anotherString.value.length) {  // 길이 비교
            // 문자 하나씩 비교하다 다르면 false
            ...
            return true;                           // 전부 같으면 true
        }
    }
    return false;
}
```

| 비교 | data1/data2 (pool) | data3/data4 (new) |
|------|:---:|:---:|
| `==` (주소) | true | **false** |
| `equals` (값) | true | **true** |

---

## 📝 정리

```
예외 응용 · API · Object
├─ 예외 응용  여러 catch로 예외별 처리 (구체적 → 포괄)
├─ API        내부(JDK) / 외부(JAR, Build Path 추가)
├─ JavaDoc    /** @author @param @return @throws */
└─ Object     toString(출력 재정의) · equals(String은 값 비교)
```

| 개념 | 한 줄 정의 |
|------|------|
| **API** | 미리 만들어진 코드의 집합 (내부/외부) |
| **JavaDoc 주석** | 배포 시 API 정보를 담는 주석 |
| **toString** | 객체 출력 형식을 정의 |
| **`==` vs equals** | 주소 비교 vs (String은) 값 비교 |

이번 글은 예외 응용부터 API 배포, Object 클래스까지 내용이 정말 많았다. 특히 `==`와 `equals`의 차이, 그리고 String이 `equals`를 값 비교로 재정의했다는 사실은 여러 번 곱씹어 확실히 내 것으로 만들 필요가 있다.
