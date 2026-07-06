---
title: "[Java] 클래스 응용 문제"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **클래스** 개념을 복습하고, 자동차 시동 시나리오로 응용 문제를 풀어본다. 정말 어려웠지만, 클래스·객체·생성자·다형성이 어떻게 맞물리는지 체감할 수 있는 문제였다.

---

## 1. 클래스 핵심 개념 복습

| 개념 | 정의 |
|------|------|
| **클래스(class)** | 추상적 개념. 공통 요소(필드)를 선언해 놓은 공간 |
| **필드(field)** | 클래스 안에 선언된 변수와 메소드 |
| **객체화(instance)** | 추상적 개념을 **구체화**하는 작업 (`new`) |
| **객체(instance)** | 구체화된 결과물 |
| **생성자(constructor)** | 필드를 메모리에 할당하고 그 주소값을 반환 (직접 만들면 초기화까지) |

```java
클래스명 객체명 = new 생성자();
```

> 💡 클래스는 "붕어빵 틀", 객체는 "구워낸 붕어빵"으로 비유된다. 필드에 접근하려면 반드시 **구체적인 객체**가 필요하다.

### 다형성 — 오버로딩(Overloading)

**매개변수의 개수나 타입이 다르면** 같은 이름의 메소드(생성자)를 여러 개 선언할 수 있다.

```java
public SuperCar(String brand, String color, int price, String pw) {   // 4개
    super();
    this.brand = brand; this.color = color;
    this.price = price; this.pw = pw;
}

public SuperCar(String brand, String color, int price) {              // 3개
    super();
    this.brand = brand; this.color = color; this.price = price;
}
```

객체 생성 시 인자를 **4개 넘기면 첫 번째**, **3개 넘기면 두 번째** 생성자가 호출된다.

---

## 2. 응용 문제 — 자동차 시동

**요구사항:**

- 시동이 이미 켜져 있으면 경고 메시지
- 시동이 이미 꺼져 있으면 경고 메시지
- 시동 켤 때 비밀번호 확인, 일치하면 켜짐
- 비밀번호 **3번 연속 오류** 시 "경찰 출동" 출력

### SuperCar 클래스 (로직)

```java
class SuperCar {
    String brand, color;
    int price;
    boolean check;          // 시동 상태
    String pw = "0000";     // 비밀번호
    int policeCnt;          // 오류 횟수

    public SuperCar() {}

    public SuperCar(String brand, String color, int price, String pw) {
        super();
        this.brand = brand; this.color = color;
        this.price = price; this.pw = pw;
    }
    public SuperCar(String brand, String color, int price) {
        super();
        this.brand = brand; this.color = color; this.price = price;
    }

    // 비밀번호 검사
    boolean checkPw(String pw) {
        return this.pw.equals(pw);
    }

    // 시동 켜기 (꺼져 있을 때만)
    boolean engineStart() {
        if (!check) { check = true; return true; }
        return false;
    }

    // 시동 끄기 (켜져 있을 때만)
    boolean engineStop() {
        if (check) { check = false; return true; }
        return false;
    }
}
```

| 메소드 | 역할 | 반환 |
|------|------|------|
| `checkPw` | 입력 pw가 저장된 pw와 같은지 | 일치하면 true |
| `engineStart` | 꺼져 있으면 켜기 | 성공 시 true |
| `engineStop` | 켜져 있으면 끄기 | 성공 시 true |

> 💡 시동 끄기도 성공하면 `true`를 반환해야 한다. "끄기 메소드가 성공했다"는 의미이기 때문이다.

### Shop 클래스 (메인)

```java
public class Shop {
    public static void main(String[] args) {
        SuperCar ferrari = new SuperCar();

        String msg = "1.시동켜기\n2.시동끄기";
        String pwMsg = "비밀번호 : ";
        int choice = 0;
        String pw = null;
        Scanner sc = new Scanner(System.in);

        while (true) {
            System.out.println(msg);
            choice = sc.nextInt();

            if (choice == 1) {                       // 시동 켜기
                if (!ferrari.check) {                 // 꺼져 있을 때만
                    System.out.println(pwMsg);
                    pw = sc.next();
                    if (ferrari.checkPw(pw)) {        // 비번 일치
                        if (ferrari.engineStart()) {
                            System.out.println("시동 켜짐");
                            ferrari.policeCnt = 0;    // 성공 시 오류 카운트 초기화
                        }
                    } else {                          // 비번 오류
                        System.out.println("비밀번호 " + ++ferrari.policeCnt + "회 오류");
                        if (ferrari.policeCnt == 3) {
                            System.out.println("경찰 출동중"); break;
                        }
                    }
                } else {
                    System.out.println("이미 시동이 켜져 있습니다.");
                }
            } else if (choice == 2) {                // 시동 끄기
                if (ferrari.engineStop()) {
                    System.out.println("시동 꺼짐");
                    break;
                } else {
                    System.out.println("이미 시동이 꺼져 있습니다.");
                }
            }
        }
    }
}
```

> ⚠️ **`policeCnt = 0` 초기화가 중요하다.** 시동을 성공적으로 켤 때 오류 횟수를 초기화하지 않으면, 평생 누적된 오류로 언젠가 3회가 되어 경찰이 출동한다.

전체 흐름을 정리하면:

```
[시동 켜기]
  꺼짐? ──► 비번 입력 ──► 일치? ──► 시동 켜짐 (policeCnt=0)
                          └ 불일치 ──► 오류 횟수++ ──► 3회면 "경찰 출동" break
  켜짐? ──► "이미 켜져 있습니다"

[시동 끄기]
  켜짐? ──► 시동 꺼짐 break
  꺼짐? ──► "이미 꺼져 있습니다"
```

---

## 📝 정리

```
클래스 응용
├─ 개념       클래스(틀) → 객체(new) → 필드 접근
├─ 생성자     필드 할당·초기화, 오버로딩으로 여러 형태
├─ 다형성     매개변수 개수/타입이 다르면 같은 이름 가능
└─ 응용       상태(check) + 검증(checkPw) + 카운트(policeCnt)
```

| 개념 | 한 줄 정의 |
|------|------|
| **클래스/객체** | 추상적 틀 / 구체화된 인스턴스 |
| **생성자** | 객체 생성 시 필드를 할당·초기화 |
| **오버로딩** | 매개변수가 다른 동명 메소드 |

처음엔 흐름을 이해하려고 몇 번을 다시 봤는지 모른다. 하지만 클래스·객체·생성자·조건문이 하나로 엮이는 걸 보며 자바의 재미를 느꼈다. 다음 시간에는 **상속**으로 돌아온다!
