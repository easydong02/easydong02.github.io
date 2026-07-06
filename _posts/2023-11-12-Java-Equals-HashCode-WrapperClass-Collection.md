---
title: "[Java] equals 재정의 hashCode Wrapper클래스 Collection"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **`equals()` 재정의**, **`hashCode()`**, 기본 타입을 객체로 감싸는 **Wrapper 클래스(박싱/언박싱)**, 그리고 **컬렉션 프레임워크의 List**를 다룬다.

---

## 1. equals() 재정의

`new`로 만든 두 객체는 내용이 같아도 **주소값이 달라 `equals`가 false**를 반환한다.

```java
Student shin = new Student(1, "신동희");
System.out.println(shin.equals(new Student(1, "신동희")));   // false
```

"학번이 같으면 같은 학생"으로 취급하려면 `equals`를 재정의한다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;              // 주소값이 같으면 당연히 true
    if (obj instanceof Student) {              // 타입 비교
        Student std = (Student) obj;           // 다운캐스팅
        if (this.num == std.num) return true;  // 학번이 같으면 같은 학생
    }
    return false;
}
```

> 💡 매개변수를 **`Object`**로 두어 모든 타입을 받고, `instanceof`로 타입을 확인한 뒤 다운캐스팅해 내용을 비교한다. 이제 학번이 같으면 true가 나온다.

---

## 2. hashCode()

`hashCode`는 간단히 말해 **주소값**과 비슷한 정수다. `Object`의 메소드 중 하나다.

```java
String data1 = "ABC";
String data2 = new String("ABC");
Random r1 = new Random();
Random r2 = new Random();

System.out.println(r1.hashCode());   // 서로 다름
System.out.println(r2.hashCode());
System.out.println(data1.hashCode()); // data1, data2는
System.out.println(data2.hashCode()); // 같은 해시코드!
```

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/1.png)

> 💡 `Random` 객체 둘은 다른 해시코드지만, `String` `"ABC"` 둘은 **같은 해시코드**다. 문자열 `"ABC"`를 선언하면 RAM의 **constant pool**에 그 주소가 생성되고, 이 값을 공유하기 때문이다.

---

## 3. Wrapper 클래스 (박싱/언박싱)

> **Wrapper 클래스란?**
> `int`, `double`, `char` 같은 **기본 자료형을 감싼 클래스 타입**. 감싸면 객체가 되어 다양한 메소드를 쓸 수 있다.

```java
// 박싱(boxing): 기본형 → 클래스형
Integer data_I = new Integer(data_i);
// 언박싱(unboxing): 클래스형 → 기본형
data_i = data_I.intValue();

// JDK 5+ 오토 박싱/언박싱 (훨씬 간단)
Integer data_I = data_i;   // auto boxing
data_i = data_I;           // auto unboxing
```

| 개념 | 방향 | 예 |
|------|------|------|
| **박싱** | 기본형 → Wrapper | `int` → `Integer` |
| **언박싱** | Wrapper → 기본형 | `Integer` → `int` |

> 💡 **왜 쓸까?** 원시 타입을 박싱하면 **다양한 메소드**를 쓸 수 있고, 캐스팅이 자유로워지며, 반드시 객체로 다뤄야 하는 상황(컬렉션 등)에서 활용할 수 있다.

---

## 4. 컬렉션 프레임워크 — List

> **컬렉션 프레임워크란?**
> 많은 데이터를 효과적으로 관리하는 **표준화된 자료구조 클래스들의 집합**. 의미 없는 데이터들이 자료구조라는 '거름망'을 거쳐 가치 있는 정보가 된다.

**List의 구현 클래스:**

| 클래스 | 특징 |
|------|------|
| `Vector` | 보안성↑ 처리량↓, 아주 옛날 방식(요즘 거의 안 씀) |
| `LinkedList` | 각 노드가 다음 노드 주소를 참조 → 삽입 빠름, 조회 느림 |
| **`ArrayList`** | **인덱스로 관리 → 조회 빠름, 실무에서 가장 많이 사용** |

### ArrayList — 배열과의 차이

| 구분 | 배열 | ArrayList |
|------|------|------|
| 크기 | **고정**(length) | **가변** (add로 자동 확장) |
| 언제 | 크기 제한이 필요할 때 | 개수를 알 수 없을 때 |

```java
ArrayList<Integer> datas = new ArrayList<>();   // <Integer> = 제네릭
datas.add(10);
System.out.println(datas.size());   // 1
System.out.println(datas);          // [10]
```

> 💡 `<E>`는 **제네릭**(타입을 지정하지 않는 기법)이다. 사용할 때 원하는 타입(Wrapper 클래스)을 지정한다. `ArrayList<Integer>`처럼.

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/2.png)

`datas`만 출력했는데 주소값이 아니라 `[10]`이 나오는 이유는, `ArrayList`가 **`toString()`을 재정의**했기 때문이다.

```java
// ArrayList의 toString() (요약)
public String toString() {
    Iterator<E> it = iterator();
    if (!it.hasNext()) return "[]";
    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (!it.hasNext()) return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

`add`를 여러 번 하면 인덱스가 자동으로 늘어난다.

```java
datas.add(10); datas.add(20); datas.add(30);
System.out.println(datas.size());   // 3
System.out.println(datas);          // [10, 20, 30]
```

![Desktop View](/assets/img/Programming-Language/Java/Equals-HashCode-WrapperClass-CollectionFramework/3.png)

---

## 📝 정리

```
equals · hashCode · Wrapper · Collection
├─ equals   내용 비교하려면 재정의 (Object 매개변수 + instanceof)
├─ hashCode 주소값 유사 정수 (문자열은 constant pool로 공유)
├─ Wrapper  기본형↔클래스형 (박싱/언박싱, auto 지원)
└─ List     ArrayList(가변, 인덱스 조회 빠름) — 실무 주력
```

| 개념 | 한 줄 정의 |
|------|------|
| **equals 재정의** | 주소 대신 내용으로 동등성 판단 |
| **박싱/언박싱** | 기본형 ↔ Wrapper 객체 변환 |
| **제네릭 `<E>`** | 사용 시 타입을 지정하는 기법 |
| **ArrayList** | 크기가 가변인 List 구현체 |

`equals` 재정의, Wrapper의 박싱, 제네릭 ArrayList는 자바 실무의 기본기다. 특히 문자열이 constant pool을 공유한다는 점, ArrayList가 `toString`을 재정의했다는 점을 알면 동작이 명확해진다.
