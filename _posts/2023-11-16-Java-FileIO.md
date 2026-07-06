---
title: "[Java] 파일입출력(FileIO)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

지난 시간에 이어 **파일 입출력(File I/O)**을 다룬다. 이번엔 파일에 텍스트를 쓰고, 특정 줄을 **수정·삭제**하는 방법을 정리한다.

> **핵심 로직**: 파일은 특정 줄만 콕 집어 바꾸기 어렵다. 그래서 **임시 저장소(temp)에 원본을 옮기며 수정/삭제하고, 완성된 temp로 원본을 덮어쓰는** 방식을 쓴다.

```
원본 파일 ──읽기(Reader)──► temp(수정/삭제 반영) ──쓰기(Writer)──► 원본 덮어쓰기
```

---

## 1. 파일에 쓰기 (BufferedWriter)

먼저 `food.txt`에 메뉴를 한 줄씩 쓴다.

```java
package day20;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.IOException;

public class FoodFileTest {
    public static void main(String[] args) throws IOException {
        BufferedWriter bw = new BufferedWriter(new FileWriter("food.txt"));
        bw.write("자장면\n");
        bw.write("짬뽕\n");
        bw.write("소고기\n");
        bw.write("닭고기\n");
        bw.close();
    }
}
```

---

## 2. 수정 — 닭고기 → 찜닭

임시 저장소 `temp`에 원본을 옮기되, 바꿀 줄만 다른 값으로 넣는다.

```java
BufferedReader br = new BufferedReader(new FileReader("food.txt"));
String line = null;
String temp = "";                         // 연결(+=)할 것이므로 "" 로 초기화
while ((line = br.readLine()) != null) {
    if (line.equals("닭고기")) {
        temp += "찜닭\n";                  // 바꿀 값을 넣고
        continue;                          // 원본 "닭고기"는 건너뜀
    }
    temp += line + "\n";                   // 나머지는 그대로
}
br.close();
```

여기까지 하면 `temp`에는 수정된 내용이 있지만, **파일은 아직 그대로**다. `temp`로 파일을 덮어써야 한다.

```java
BufferedWriter bw2 = new BufferedWriter(new FileWriter("food.txt"));
bw2.write(temp);
bw2.close();
```

![Desktop View](/assets/img/Programming-Language/Java/FileIO/1.png)

정상적으로 수정됐다.

---

## 3. 삭제 — 소고기 제거

삭제는 수정보다 더 간단하다. **바꿀 값을 넣지 않고 그냥 `continue`**만 하면 그 줄이 temp에서 빠진다.

```java
BufferedReader br = new BufferedReader(new FileReader("food.txt"));
String line = null;
String temp = "";
while ((line = br.readLine()) != null) {
    if (line.equals("소고기")) {
        continue;                          // temp에 넣지 않고 건너뜀 = 삭제
    }
    temp += line + "\n";
}
br.close();

BufferedWriter bw2 = new BufferedWriter(new FileWriter("food.txt"));
bw2.write(temp);
bw2.close();
```

| 작업 | 매칭 시 동작 |
|------|------|
| **수정** | `temp += "새값\n"; continue;` (바꿀 값 넣고 원본 건너뜀) |
| **삭제** | `continue;` (아무것도 안 넣고 건너뜀) |

![Desktop View](/assets/img/Programming-Language/Java/FileIO/2.png)

정상적으로 삭제됐다.

---

## 📝 정리

```
파일 I/O 수정·삭제 패턴
├─ 쓰기    BufferedWriter.write()
├─ 읽기    BufferedReader.readLine()
├─ 수정    매칭 줄 → 새 값 넣고 continue
├─ 삭제    매칭 줄 → 그냥 continue (temp에서 제외)
└─ 마무리  temp를 원본에 덮어쓰기
```

핵심은 **"임시 저장소에 옮기며 가공한 뒤 덮어쓴다"**는 패턴이다. 이 하나의 로직으로 수정과 삭제를 모두 처리할 수 있다.

이로써 2달간의 자바 공부가 마무리됐다. 새로운 세상을 경험한 시간이었다. 앞으로는 국비교육에서 배운 것들과 느낀 점들을 이어서 포스팅하려 한다. 그동안 봐주셔서 감사합니다!
