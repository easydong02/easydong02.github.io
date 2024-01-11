---
title: "[Java] 직렬화 / 역직렬화"
date: 2024-01-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java, serializable]
render_with_liquid: false
---

# 직렬화 / 역직렬화

## ✅**Java 직렬화와 역직렬화: 객체를 파일로 저장하자!**

---

안녕하세요! 오늘은 Java에서 많이 사용되는 직렬화(Serialization)와 역직렬화(Deserialization)에 대해 살펴보겠습니다. 이 두 기술은 객체를 파일로 저장하고 다시 읽어오는 등의 작업을 할 때 유용하게 사용됩니다.

## ✅**직렬화란 무엇인가?**

---

직렬화는 객체를 바이트 스트림으로 변환하는 과정입니다. 이를 통해 객체의 상태를 저장하거나 네트워크를 통해 객체를 전송할 수 있습니다. Java에서는 **`Serializable`** 인터페이스를 구현함으로써 직렬화를 지원합니다.

### **예시 코드**

```java
javaCopy code
import java.io.*;

public class SerializationExample {
    public static void main(String[] args) {
        // 직렬화할 객체 생성
        Person person = new Person("John Doe", 25);

        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            // 객체를 파일로 저장
            oos.writeObject(person);
            System.out.println("직렬화 완료");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// 직렬화 가능한 클래스
class Person implements Serializable {
    private String name;
    private int age;

    // 생성자, 게터, 세터 등...

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

## ✅**역직렬화란 무엇인가?**

---

역직렬화는 직렬화된 바이트 스트림을 객체로 변환하는 과정입니다. 저장된 파일이나 네트워크를 통해 전송된 데이터를 읽어와서 원래의 객체로 복원합니다.

### **예시 코드**

```java
javaCopy code
import java.io.*;

public class DeserializationExample {
    public static void main(String[] args) {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.ser"))) {
            // 파일에서 객체 역직렬화
            Person person = (Person) ois.readObject();
            System.out.println("역직렬화 완료");
            System.out.println("복원된 객체: " + person);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

## 📌**주의사항과 고려할 점**

---

1. **버전 관리**: 클래스의 구조가 변경되면 직렬화된 데이터를 역직렬화할 때 문제가 발생할 수 있으므로 버전 관리에 주의해야 합니다.
2. **보안**: 직렬화된 데이터는 가독성이 떨어지고, 수정이 쉽지 않아 보안상 이점이 있을 수 있습니다.
3. **직렬화의 성능**: 대용량의 데이터를 직렬화할 때는 성능 이슈에 주의해야 합니다.
