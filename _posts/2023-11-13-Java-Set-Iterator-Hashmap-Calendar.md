---
title: "[Java] Set Iterator Hashmap Calendar"
date: 2023-11-13 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
---

## 📌 들어가며

이번 글에서는 컬렉션 프레임워크의 **Set**, 순서를 부여하는 **Iterator**, 키-값 쌍인 **HashMap**, 향상된 for문 **forEach**, 그리고 날짜 클래스 **Date·Calendar**를 정리한다.

---

## 1. Set — 중복 없는 집합

> **HashSet**: 중복 원소를 포함할 수 없고, **순서가 없다**. 값의 유무 검사에 특화되어 있으며, 해시 코드로 검사해 속도가 좋다.

```java
HashSet<String> bloodTypeSet = new HashSet<>();
bloodTypeSet.add("A");
bloodTypeSet.add("B");
bloodTypeSet.add("O");
bloodTypeSet.add("AB");
bloodTypeSet.add("AB");   // 여러 번 넣어도
bloodTypeSet.add("AB");   // 하나의 AB로 취급
System.out.println(bloodTypeSet);
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/1.png)

`AB`를 여러 번 넣어도 하나만 저장된다. 그런데 **인덱스가 없는데 어떻게 값을 꺼낼까?** 답은 `toString`에 숨은 **Iterator**에 있다.

---

## 2. Iterator — 순서 부여

> **Iterator**: 순서가 없는 객체에 **순서를 부여**하거나, 순서 방식을 iterator식으로 바꿀 때 사용. `hasNext()`로 다음 값 유무를 검사하고 `next()`로 값을 꺼낸다.

```java
Iterator<String> iter = bloodTypeSet.iterator();   // Set에 순서 부여
while (iter.hasNext()) {
    System.out.println(iter.next());
}
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/2.png)

| 메소드 | 역할 |
|------|------|
| `iterator()` | 순서를 부여한 Iterator 반환 |
| `hasNext()` | 다음 값이 있는지 검사 |
| `next()` | 다음 값을 꺼냄 |

> 💡 `HashSet`의 `toString()` 내부에도 `Iterator<E> it = iterator();`(`this.iterator()`의 생략)가 쓰인다. 그래서 인덱스 없는 Set도 출력할 수 있었던 것이다.

---

## 3. HashMap — 키·값 쌍

> **HashMap**: **Key-Value 한 쌍**으로 저장하며 검색이 목적. **Key는 중복 불가(Set)**, Value는 중복 허용. 같은 Key를 넣으면 Value가 최신 값으로 갱신된다.

```java
HashMap<String, Integer> fruitMap = new HashMap<>();
fruitMap.put("사과", 3000);
fruitMap.put("배", 4000);
fruitMap.put("수박", 8000);
fruitMap.put("자두", 800);

System.out.println(fruitMap.size());     // 4 (한 쌍 = 1)
System.out.println(fruitMap);            // {사과=3000, ...}
System.out.println(fruitMap.get("배"));  // 4000 (키로 값 조회)
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/3.png)

| 메소드 | 역할 |
|------|------|
| `put(k, v)` | 키-값 쌍 저장 |
| `get(k)` | 키로 값 조회 |
| `size()` | **쌍의 개수** (8이 아니라 4) |
| `keySet()` | 키들만 Set으로 |
| `values()` | 값들만 |

**키 분리** — `keySet()`으로 키만 뽑아 Iterator로 순회한다.

```java
Iterator<String> fruitName = fruitMap.keySet().iterator();
while (fruitName.hasNext()) {
    System.out.println(fruitName.next());
}
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/4.png)

---

## 4. forEach (향상된 for문)

일반 for문은 `초기식; 조건식; 증감식`이지만, 향상된 for문은 **콜론(`:`)**으로 이어지고 **반복 횟수를 알아서** 정한다.

```java
for (자료형 변수명 : 배열/컬렉션) {
    반복할 문장;
}
```

```java
String[] arData = {"안녕", "하세요"};
for (String data : arData) {   // arData의 값이 하나씩 data로
    System.out.println(data);
}
```

**2차원 배열**은 이중으로 쓴다. (바깥은 행, 안쪽은 열)

```java
int[][] arrData = {
    {10, 25, 30}, {11, 24, 35}, {12, 23, 40}, {13, 22, 45}, {14, 21, 50}
};
int num = 0;
for (int[] arData : arrData) {          // 행 단위
    num++;
    int total = 0;
    for (int data : arData) {           // 그 행의 열들
        total += data;
    }
    double avg = (double) total / arData.length;
    avg = Double.parseDouble(String.format("%.2f", avg));
    System.out.println(num + "번 학생 총 점: " + total + "점");
    System.out.println(num + "번 학생 평균: " + avg + "점");
}
```

**HashMap의 값 평균**도 forEach로 간단히 구한다.

```java
int sum = 0;
for (int data : fruitMap.values()) {
    sum += data;
}
double avg = (double) sum / fruitMap.values().size();
avg = Double.parseDouble(String.format("%.2f", avg));
System.out.println(avg);
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/5.png)

---

## 5. Date와 Calendar

### Date

```java
Date today = new Date();
System.out.println(today);   // 현재 시간 (가독성 낮음)
```

가독성이 떨어져 **`SimpleDateFormat`**으로 포맷한다.

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy년 MM월 dd일");
System.out.println(sdf.format(today));
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/7.png)

### Calendar

> `Calendar`는 **추상 클래스**라 `new`를 쓰지 않는다. 달력·시간은 각자 인스턴스화하면 안 되고 **하나만 존재**해야 하기 때문. 대신 `getInstance()`로 얻는다.

```java
Calendar cal = Calendar.getInstance();   // new 대신 getInstance()
cal.set(2021, 11, 04);                    // 주의: 11 = 12월!

SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
System.out.println(sdf.format(cal.getTime()));  // Calendar → Date 변환
System.out.println(cal.get(Calendar.MONTH) + 1); // 월도 +1 필요
```

![Desktop View](/assets/img/Programming-Language/Java/Set-Iterator-Hashmap-Date-Calendar/8.png)

> ⚠️ **월은 0부터 시작**한다. 12월을 넣으려면 `set(..., 11, ...)`, 월을 꺼낼 때도 **`+1`**을 해야 한다. 또 `format()`에는 Date만 들어가므로 `cal.getTime()`으로 Calendar를 Date로 변환한다.

---

## 📝 정리

```
Set · Iterator · HashMap · forEach · Date
├─ HashSet   중복X, 순서X (유무 검사 특화)
├─ Iterator  순서 부여, hasNext()/next()
├─ HashMap   Key(중복X)-Value(중복O), get으로 조회
├─ forEach   for(타입 v : 컬렉션) — 반복 자동
└─ Date/Calendar  SimpleDateFormat 포맷, Calendar는 월 0부터
```

| 개념 | 한 줄 정의 |
|------|------|
| **HashSet** | 중복 없는 집합 |
| **Iterator** | 순서를 부여해 순회 |
| **HashMap** | 키-값 쌍 저장·검색 |
| **forEach** | 콜론 기반 향상된 for문 |
| **Calendar** | 추상 클래스, getInstance()로 사용(월 0부터) |

Set·Map·Iterator·forEach는 자바 컬렉션의 기본기다. 특히 **HashMap의 Key가 Set(중복 불가)**이라는 점, **Calendar의 월이 0부터**라는 함정을 기억하면 실수를 줄일 수 있다.
