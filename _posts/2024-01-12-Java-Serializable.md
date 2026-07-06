---
title: "[Java] 직렬화 / 역직렬화"
date: 2024-01-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, serializable]
render_with_liquid: false
---

## 📌 들어가며

메모리 위의 객체는 프로그램이 종료되면 사라진다. 그런데 이 객체를 **파일로 저장**하거나 **네트워크로 전송**하려면 어떻게 해야 할까? 객체를 바이트의 흐름으로 바꿔야 한다. 이 변환 과정이 **직렬화(Serialization)**이고, 되돌리는 것이 **역직렬화(Deserialization)**다.

```
[직렬화]    객체  ──►  바이트 스트림  ──►  파일 / 네트워크
[역직렬화]  파일 / 네트워크  ──►  바이트 스트림  ──►  객체 복원
```

| 용어 | 정의 |
|------|------|
| **직렬화** | 객체 → 바이트 스트림 (저장·전송 가능한 형태로) |
| **역직렬화** | 바이트 스트림 → 객체 (원래 상태로 복원) |

---

## 1. 직렬화 — 객체를 파일로 저장

Java에서는 **`Serializable` 인터페이스를 구현**하면 직렬화가 지원된다. 저장에는 `ObjectOutputStream.writeObject()`를 쓴다.

```java
import java.io.*;

public class SerializationExample {
    public static void main(String[] args) {
        // 직렬화할 객체 생성
        Person person = new Person("John Doe", 25);

        try (ObjectOutputStream oos =
                 new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            oos.writeObject(person);          // 객체를 파일로 저장
            System.out.println("직렬화 완료");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// 직렬화 가능한 클래스 → Serializable 구현
class Person implements Serializable {
    private String name;
    private int age;

    // 생성자, 게터, 세터 등...

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}
```

> 💡 핵심은 `class Person implements Serializable`이다. 이 인터페이스는 메소드가 없는 **마커 인터페이스**로, "이 클래스는 직렬화해도 된다"는 표시 역할만 한다.

---

## 2. 역직렬화 — 파일에서 객체 복원

저장된 파일을 `ObjectInputStream.readObject()`로 읽어 객체로 되돌린다.

```java
import java.io.*;

public class DeserializationExample {
    public static void main(String[] args) {
        try (ObjectInputStream ois =
                 new ObjectInputStream(new FileInputStream("person.ser"))) {
            Person person = (Person) ois.readObject();   // 형변환 필요
            System.out.println("역직렬화 완료");
            System.out.println("복원된 객체: " + person);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

> ⚠️ `readObject()`는 `Object`를 반환하므로 원래 타입으로 **형변환**해야 하며, `ClassNotFoundException`도 함께 처리해야 한다.

---

## ⚠️ 주의사항과 고려할 점

| 항목 | 내용 |
|------|------|
| **버전 관리** | 클래스 구조가 바뀌면 기존 직렬화 데이터의 역직렬화에서 문제 발생 → `serialVersionUID` 관리 필요 |
| **보안** | 직렬화 데이터는 가독성이 떨어지지만, 신뢰할 수 없는 데이터의 역직렬화는 보안 취약점이 될 수 있음 |
| **성능** | 대용량 데이터 직렬화 시 성능 이슈 주의 |

---

## 📝 정리

```
직렬화 / 역직렬화
├─ 직렬화    객체 → 바이트 스트림   writeObject()  (Serializable 구현)
├─ 역직렬화  바이트 스트림 → 객체   readObject()   (형변환 필요)
└─ 주의      버전(serialVersionUID) · 보안 · 성능
```

| 개념 | 한 줄 정의 |
|------|------|
| **Serializable** | 직렬화를 허용하는 마커 인터페이스 |
| **writeObject / readObject** | 객체를 저장 / 복원하는 메소드 |
| **serialVersionUID** | 클래스 버전을 식별해 호환성을 관리 |

직렬화는 "메모리 속 객체를 파일이나 네트워크로 실어 나르는" 기술이다. `Serializable`만 구현하면 되는 간단함이 매력이지만, 버전 관리와 보안은 반드시 신경 써야 한다.
