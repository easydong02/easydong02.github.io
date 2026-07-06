---
title: "[Java] 오버라이딩(hashCode, equals)"
date: 2024-02-29 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, override, hashcode, equals]
render_with_liquid: false
---

## 📌 들어가며

`new`로 만든 두 객체는 내용이 같아도 기본적으로 "다른 객체"로 취급된다. 그런데 "이름과 나이가 같으면 같은 사람"처럼 **내용 기반의 동등성**을 정의하고 싶을 때가 있다. 이때 재정의(오버라이딩)해야 하는 것이 **`equals()`**와 **`hashCode()`**다. 특히 `HashMap`, `HashSet` 같은 해시 기반 자료구조에서 이 둘은 반드시 짝으로 다뤄야 한다.

> **왜 둘을 함께?**
> `equals()`로 같다고 판단되는 두 객체는 **반드시 같은 `hashCode()`**를 반환해야 한다. 이 규칙이 깨지면 해시 자료구조에서 값을 못 찾는 버그가 생긴다.

---

## 1. hashCode() 메소드

**`hashCode()`**는 객체의 해시 코드(정수)를 반환한다. 해시 테이블에 객체를 저장할 때 **어느 버킷에 넣을지**를 결정하는 값이다.

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + field1.hashCode();
    result = 31 * result + field2.hashCode();
    return result;
}
```

| 규칙 | 설명 |
|------|------|
| **일관성** | 같은 객체는 여러 번 호출해도 항상 같은 값 |
| **equals 일치** | `equals()`가 true인 두 객체는 **같은 해시 코드** 반환 |

> 💡 `31`을 곱하는 것은 관례다. 홀수이면서 소수라 해시 충돌을 줄이는 데 유리하고, `31 * i`가 `(i << 5) - i`로 최적화되기 때문이다.

---

## 2. equals() 메소드

**`equals()`**는 두 객체가 **동등한지**를 판단한다. 내용이 같은지를 비교한다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;                          // 같은 참조면 true
    if (obj == null || getClass() != obj.getClass())       // null·타입 검사
        return false;
    MyClass other = (MyClass) obj;
    return Objects.equals(field1, other.field1) &&
           Objects.equals(field2, other.field2);
}
```

`equals()`를 재정의할 때는 다음 **5가지 규약**을 지켜야 한다.

| 규약 | 의미 |
|------|------|
| **반사성(reflexive)** | `x.equals(x)`는 항상 true |
| **대칭성(symmetric)** | `x.equals(y)`가 true면 `y.equals(x)`도 true |
| **추이성(transitive)** | `x=y`, `y=z`면 `x=z`도 성립 |
| **일관성(consistent)** | 상태가 안 바뀌면 결과도 항상 동일 |
| **null 안전(null-safe)** | `null` 인자에는 항상 false |

---

## 3. 오버라이딩 예시

실무에서는 `Objects.hash()`와 `Objects.equals()`를 쓰면 간결하고 안전하다.

```java
public class Person {
    private String name;
    private int age;

    // constructors, getters, setters

    @Override
    public int hashCode() {
        return Objects.hash(name, age);        // 여러 필드를 한 번에
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age &&
               Objects.equals(name, person.name);
    }
}
```

위 예시는 `name`과 `age` 필드를 기준으로 두 메소드를 재정의했다. 이제 두 `Person`의 이름·나이가 같으면 **동등한 객체**로 취급되고, `HashMap`의 키로도 올바르게 동작한다.

---

## 📝 정리

```
equals() / hashCode() 오버라이딩
├─ hashCode  객체 → 정수(버킷 결정),  Objects.hash(...)
├─ equals    내용 동등성 판단,        Objects.equals(...)
├─ 핵심 규칙  equals가 true면 hashCode도 반드시 같아야 함
└─ equals 규약  반사·대칭·추이·일관·null 안전
```

| 개념 | 한 줄 정의 |
|------|------|
| **hashCode()** | 해시 자료구조에서 버킷을 결정하는 정수 |
| **equals()** | 두 객체의 내용 동등성 판단 |
| **핵심 계약** | equals가 같으면 hashCode도 같아야 함 |

`equals()`와 `hashCode()`는 **항상 짝으로** 재정의해야 한다. 둘 중 하나만 재정의하면 `HashMap`·`HashSet`에서 예상치 못한 버그가 생기니, `Objects.hash`/`Objects.equals`를 활용해 함께 정의하자.
