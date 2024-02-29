---
title: "[Java] 오버라이딩(hashCode, equals)"
date: 2024-02-29 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java, override, hashcode, equals]
render_with_liquid: false
---

# **오버라이딩 - hashCode(), equals()**

자바에서 객체지향 프로그래밍을 하다보면 **`hashCode()`**와 **`equals()`** 메서드를 오버라이딩(재정의)해야 하는 경우가 있습니다. 이 두 메서드는 객체의 동등성(equality)을 판단하고, 객체를 해시맵(HashMap) 등의 자료구조에 사용할 때 중요한 역할을 합니다. 이번 포스팅에서는 **`hashCode()`**와 **`equals()`** 메서드의 개념을 살펴보고, 어떻게 오버라이딩하는지에 대해 알아보겠습니다.

## ✅**hashCode() 메서드**

---

**`hashCode()`** 메서드는 객체의 해시 코드를 반환합니다. 해시 코드는 객체가 해시 테이블에 저장될 때 사용되는 정수 값입니다. 동일한 객체에 대해 여러 번 호출되더라도 일관된 해시 코드를 반환해야 합니다.

```java
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + field1.hashCode();
    result = 31 * result + field2.hashCode();
    return result;
}

```

**`hashCode()`**를 오버라이딩할 때 주의할 점은 동일한 객체는 항상 같은 해시 코드를 반환해야 한다는 것입니다. 또한 서로 다른 객체지만 **`equals()`** 메서드로 비교했을 때 동등하다고 판단되는 객체는 동일한 해시 코드를 반환해야 합니다.

## ✅**equals() 메서드**

---

**`equals()`** 메서드는 두 객체가 동등한지 여부를 판단합니다. 객체의 내용이 같은지를 비교하는 데 사용됩니다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) {
        return true;
    }
    if (obj == null || getClass() != obj.getClass()) {
        return false;
    }
    MyClass other = (MyClass) obj;
    return Objects.equals(field1, other.field1) &&
           Objects.equals(field2, other.field2);
}

```

**`equals()`**를 오버라이딩할 때는 다음을 주의해야 합니다.

- 반사성(reflexive): **`x.equals(x)`**는 항상 true를 반환해야 합니다.
- 대칭성(symmetric): **`x.equals(y)`**가 true이면 **`y.equals(x)`**도 true여야 합니다.
- 추이성(transitive): **`x.equals(y)`**가 true이고 **`y.equals(z)`**가 true이면 **`x.equals(z)`**도 true여야 합니다.
- 일관성(consistent): 객체의 상태가 변경되지 않는 한 **`equals()`** 호출 결과는 항상 같아야 합니다.
- null 검사(null-safe): **`null`**을 인자로 받았을 때는 항상 **`false`**를 반환해야 합니다.

## ✅**오버라이딩 예시**

---

```java
public class Person {
    private String name;
    private int age;

    // constructors, getters, setters

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
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

위의 예시에서는 **`hashCode()`**와 **`equals()`**를 문자열 **`name`**과 정수 **`age`** 필드를 기반으로 오버라이딩하고 있습니다.