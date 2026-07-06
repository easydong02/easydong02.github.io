---
title: "[Java] 클래스 메소드 문제"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

지난 시간에 배운 메소드로 문제를 풀어보고, **클래스·생성자·this**를 복습한다. 특히 "메소드에서 값을 2개 이상 돌려주려면?", "왜 생성자에 `this`가 필요한가?" 같은 실전 감각을 잡는 것이 목표다.

---

## 1. 문제 ① — 최대·최소를 한 번에 구하기

리턴값은 **1개**만 돌려줄 수 있다. 그런데 최대·최소 **2개**를 돌려주려면? **배열(주소값)**을 활용한다.

```java
void getMaxAndMin(int[] nums, int[] result) {
    int max = nums[0];
    int min = nums[0];
    for (int i = 1; i < nums.length; i++) {
        if (max < nums[i]) max = nums[i];
        if (min > nums[i]) min = nums[i];
    }
    result[0] = max;   // 결과를 배열에 담아 '외부'로 전달
    result[1] = min;
}
```

> 💡 **왜 매개변수가 배열일까?** 단순 값 변수는 한 지역에서 선언·사용되면 다른 지역에서 쓸 수 없다. 하지만 배열은 **주소값**을 주고받으므로, 다른 지역(메소드)에서 그 주소로 찾아가 값을 수정하면 **원본에도 반영**된다. 그래서 `void`로 두고 배열에 결과를 담는다.

```java
int[] result = new int[2];
int[] nums = {3, 2, 5, 7, 8};

m.getMaxAndMin(nums, result);

System.out.println("최대값 : " + result[0]);
System.out.println("최소값 : " + result[1]);
```

![Desktop View](/assets/img/Programming-Language/Java/Class-Method/1.png)

다른 지역의 메소드에서 값을 수정했는데 메인에서 출력된다. 배열이 **고유한 주소**를 갖고, 그 주소로 찾아가 값을 넣었기 때문이다.

---

## 2. 문제 ② — 한글 숫자를 정수로 (예: 구일공삼 → 9103)

```java
long changeToInteger(String hangle) {
    String hangleOrigin = "공일이삼사오육칠팔구";
    String result = "";
    for (int i = 0; i < hangle.length(); i++) {
        result += hangleOrigin.indexOf(hangle.charAt(i));   // 문자 → 위치(숫자)
    }
    return Long.parseLong(result);
}
```

`charAt`과 `indexOf`는 방향이 반대다.

| 메소드 | 입력 → 출력 |
|------|------|
| `charAt(위치)` | 위치 → 그 위치의 문자 |
| `indexOf(문자)` | 문자 → 처음 만난 위치(숫자) |

> 💡 리턴타입이 `int`가 아니라 **`long`**인 이유: 자릿수가 커지면 4byte `int`를 넘어가므로 8byte `long`을 썼다.

![Desktop View](/assets/img/Programming-Language/Java/Class-Method/2.png)

---

## 3. 클래스 복습

클래스는 처음엔 어려웠지만, 두 가지 관점으로 보면 이해가 쉽다.

| 관점 | 의미 |
|------|------|
| **하나의 타입** | 필드를 한 번 선언하면 여러 곳에서 사용. 단, 접근하려면 그 타입의 변수(객체)가 필요 |
| **주어(대문자)** | 영어 문장의 첫 단어처럼 대문자로 시작 |

> 💡 `Monkey.eat("바나나")` — `eat`은 소괄호가 있으니 **메소드(동사)**, `Monkey`는 **클래스(주어, 명사)**. 합치면 하나의 영어 문장이 된다! 클래스·메소드·변수의 명명 규칙이 하나로 연결되는 순간이다.

### 생성자와 this

```java
class Car {
    String brand, color;
    int price;

    // 직접 생성자: 매개변수로 필드를 초기화
    Car(String brand, String color, int price) {
        this.brand = brand;   // this.필드 = 매개변수
        this.color = color;
        this.price = price;
    }
}

// 사용
Car mom = new Car("Benz", "Black", 8000);
```

| 개념 | 설명 |
|------|------|
| **기본 생성자** | 클래스 선언 시 자동 생성. 직접 생성자를 만들면 사라짐 |
| **직접 생성자** | 필드를 매개변수로 받아 **초기화까지** 수행 |
| **`this`** | 현재 접근한 객체 자신. `this.필드`로 매개변수와 구분 |

> 💡 **왜 `this`가 필요한가?** 매개변수 이름과 필드 이름이 같으면 `brand = brand;`가 되어 구분이 안 된다. `this.brand`는 "이 객체의 필드", `brand`는 "매개변수"를 뜻하므로 둘을 명확히 구분한다. 여러 객체가 접근할 때, 지금 접근한 객체의 주소값이 `this`에 자동으로 담긴다.

---

## 📝 정리

```
클래스·메소드 문제
├─ 다중 리턴   배열(주소값)로 여러 값을 외부에 전달
├─ charAt/indexOf  위치→문자 / 문자→위치 (반대 방향)
├─ 클래스     하나의 타입 + 주어(대문자)
└─ 생성자     직접 생성자로 초기화, this로 필드-매개변수 구분
```

| 개념 | 한 줄 정의 |
|------|------|
| **배열로 다중 리턴** | 주소값을 넘겨 여러 결과를 전달 |
| **this** | 현재 객체 자신을 가리키는 참조 |
| **직접 생성자** | 필드를 초기화하는 사용자 정의 생성자 |

클래스는 양이 많고 복잡하지만, "타입이자 주어"라는 관점과 `this`의 역할을 잡으면 나머지가 술술 풀린다. 앞으로도 클래스는 계속 등장할 예정이다.
