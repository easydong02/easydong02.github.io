---
title: "[Java] 함수 2차원배열"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 저번 시간에 이은 **2차원 배열**을 살짝 살펴보고, **메소드(method)**에 대해 정리한다.

---

## 1. 2차원 배열

**2차원 배열**은 쉽게 말하면 **배열 안의 배열**, 즉 행과 열의 개념이다. 실제로는 1차원 배열의 한 칸이 다시 열의 개수만큼 쪼개진다.

> 💡 `arrData[3][3]`에서 `arrData`의 주소는 `arrData[0][0]`이다. 여기서 `+1`을 하면 `arrData[0][1]`이 아니라 **`arrData[1][0]`**이 된다. (행 단위로 이동)

**선언 방법:**

```java
자료형[][] 배열명 = { {값1, 값2, ...}, ... };   // 값으로 초기화
자료형[][] 배열명 = new 자료형[행][열];          // 크기로 선언
```

> ⚠️ 2차원 배열부터는 메모리 낭비가 심해 선호되지 않는다고 한다. 그래도 하나 만들어 보자.

```java
int[][] arrData = {{2, 4, 6}, {1, 3, 5}};

// 전체 길이 = 행 길이 × 열 길이
int length = arrData.length * arrData[0].length;

for (int i = 0; i < arrData.length; i++) {          // i = 행
    for (int j = 0; j < arrData[i].length; j++) {   // j = 그 행의 열
        System.out.println(arrData[i][j]);
    }
}
```

| 표현 | 의미 |
|------|------|
| `arrData.length` | **행의 개수** (여기선 2) |
| `arrData[i].length` | **i행이 가진 열의 개수** |
| `arrData.length * arrData[0].length` | 전체 원소 개수 |

> 💡 `arrData[i].length`로 각 행의 열 개수를 따로 구하는 이유는, **정방 배열이 아닐 때**(행마다 열 개수가 다를 때)도 안전하게 순회하기 위해서다.

---

## 2. 메소드 (method)

수학의 함수와 비슷하다.

```
f(x) = 2x + 1
│  │     └── 리턴값
│  └── 매개변수
└── 메소드명
```

**선언 형식:**

```java
리턴타입 메소드명(자료형 매개변수1, ...) {
    실행할 문장;
    return 리턴값;
}
```

| 구성 요소 | 설명 |
|------|------|
| **리턴타입** | 리턴값의 타입. 없으면 `void` |
| **메소드명** | 보통 **동사** (변수는 명사, 메소드는 동작이므로) |
| **매개변수** | 외부에서 전달받을 값의 타입에 맞춰 선언 |
| **return** | 결과값을 호출한 곳으로 반환 |

> 💡 **선언인지 사용인지 구분하는 팁**
> - 메소드 선언은 반드시 **메소드 밖**에서 → **중괄호 `{`가 열려 있으면 선언**
> - 메소드 사용(호출)은 반드시 **메소드 안**에서 → **중괄호가 없으면 사용**

---

## 3. 예제

```java
public class MethodTest {

    // main 밖에서 f 메소드 '선언'
    int f(int x) {
        int result = 2 * x + 1;
        return result;
    }

    public static void main(String[] args) {
        MethodTest m = new MethodTest();   // 객체 생성 후 사용

        System.out.println((double)(m.f(3)));       // 7 → 7.0
        System.out.println((char)(m.f(2) + 'A'));   // 5 + 'A' → 'F'
    }
}
```

![Desktop View](/assets/img/Programming-Language/Java/Method/1.png)

> ⚠️ C에서는 함수를 바로 호출했지만, **Java에서는 `MethodTest m = new MethodTest();`로 객체를 만든 뒤 `m.f(값)`으로 사용**한다. (정확한 이유는 객체지향을 배우면 이어서 정리한다.)

- 위: `f(3) = 7`을 `double`로 강제 형변환 → `7.0`
- 아래: `f(2) = 5`, `'A' + 5` → 'A'에서 5칸 뒤 알파벳인 **'F'** 출력

---

## 📝 정리

```
2차원 배열 & 메소드
├─ 2차원 배열   배열 안의 배열, arrData[행][열]
│               .length=행수, arrData[i].length=그 행의 열수
└─ 메소드       리턴타입 이름(매개변수){ ...; return; }
                { 열려있으면 선언, 없으면 호출
                Java는 객체 생성 후 m.메소드() 로 사용
```

| 개념 | 한 줄 정의 |
|------|------|
| **2차원 배열** | 행·열 구조의 배열(배열 안의 배열) |
| **메소드** | 리턴값을 갖는 동작 단위 (수학의 함수와 유사) |
| **선언 vs 호출** | 중괄호가 있으면 선언, 없으면 호출 |

메소드는 앞으로 배울 객체지향의 기본 단위다. 특히 Java는 (C와 달리) **객체를 만들어야 인스턴스 메소드를 호출**할 수 있다는 점을 기억해두자.
