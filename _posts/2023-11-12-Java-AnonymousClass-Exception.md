---
title: "[Java] 익명클래스 예외처리"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

## 📌 들어가며

이번 글에서는 **익명 클래스(Anonymous inner class)**와 **예외 처리(Exception)**를 다룬다. 익명 클래스는 스타벅스 지점 예제로, 예외 처리는 배열 인덱스 오류로 실습한다.

---

## 1. 익명 클래스

> **익명 클래스란?**
> **이름이 없는 클래스.** 구현되지 않은 필드를 **일회성으로** 구현하기 위해 그 자리에서 즉석으로 만드는 클래스다.

스타벅스 인터페이스와 클래스를 준비하자.

```java
public interface Form {
    public String[] getMenu();
    public void sell(String order);
}
```

```java
public class Starbucks {
    Scanner sc = new Scanner(System.in);
    public void register(Form form) {
        String[] arMenu = form.getMenu();
        System.out.println("======메뉴판======");
        for (int i = 0; i < arMenu.length; i++) {
            System.out.println((i + 1) + ". " + arMenu[i]);
        }
        if (form instanceof FormAdapter) {      // 어댑터면 무료 나눔
            System.out.println("무료 나눔중...");
        } else {
            form.sell(sc.next());               // 아니면 판매
        }
    }
}
```

```java
// sell을 강제 구현하지 않아도 되게 하는 Adapter
public abstract class FormAdapter implements Form {
    @Override public String[] getMenu()      { return null; }
    @Override public void sell(String order)  {}
}
```

### 지점을 익명 클래스로 즉석 구현

```java
public class Road {
    public static void main(String[] args) {
        // 잠실점: sell 없이 getMenu만 (무료 나눔)
        Starbucks jamsil = new Starbucks();
        jamsil.register(new FormAdapter() {          // ★ 익명 클래스
            @Override public String[] getMenu() {
                return new String[]{"아메리카노", "딸기주스", "에스프레소"};
            }
        });

        // 강남점: sell + getMenu 모두 구현
        Starbucks gangnam = new Starbucks();
        gangnam.register(new Form() {                // ★ 익명 클래스
            @Override public void sell(String order) {
                String[] arMenu = getMenu();
                for (int i = 0; i < arMenu.length; i++) {
                    if (arMenu[i].equals(order)) {
                        System.out.println(order + " 주문완료");
                        break;
                    }
                }
            }
            @Override public String[] getMenu() {
                return new String[]{"아메리카노", "카페라떼", "에스프레소"};
            }
        });
    }
}
```

> 💡 **매개변수 자리에 긴 코드**가 들어간 것이 익명 클래스다. `register(...)`를 호출하면서, 그 인자를 **그 자리에서 일회용으로 구현**한 것이다. 잠실점은 Adapter를 써서 `getMenu`만, 강남점은 `Form`을 직접 써서 둘 다 구현했다.

![Desktop View](/assets/img/Programming-Language/Java/AnonymousClass-Exception/1.png)

`instanceof FormAdapter` 분기 덕분에, 어댑터로 넘긴 잠실점은 **"무료 나눔중..."**, 강남점은 실제 판매가 동작한다.

---

## 2. 예외 처리 (Exception)

> **예외 처리란?**
> 실행 중 발생하는 에러를 잡아, **프로그램이 중간에 멈추지 않고** 정상 흐름을 이어가게 하는 방식.

**문법:**

```java
try {
    // 오류가 발생할 수 있는 문장
} catch (예외이름 e) {
    // 오류 발생 시 실행할 문장
} catch (예외이름 e) {
    // 또 다른 예외 처리
} finally {
    // 오류 여부와 무관하게 무조건 실행 (주로 자원 닫기)
}
```

| 구성 | 역할 |
|------|------|
| `try` | 오류 가능성이 있는 코드 |
| `catch` | 특정 예외가 나면 처리 |
| `finally` | 무조건 실행 (외부 장치 닫기 등) |

**사용 이유:** ① 제어문으로 처리 못 하는 경우가 있고, ② 프로그램 강제 종료(팅김)를 막기 위해서다.

### 실습 — 배열 인덱스 오류

```java
int[] arData = new int[5];
try {
    arData[10] = 10;          // 5칸 배열에 인덱스 10 → 오류
} catch (Exception e) {
    System.out.println(e);    // 어떤 오류인지 출력
}
```

![Desktop View](/assets/img/Programming-Language/Java/AnonymousClass-Exception/2.png)

> 💡 자바는 에러를 "미지의 오작동"으로 두지 않고 **타입으로 정의**해놓았다. 모든 예외의 부모는 **`Exception`**이다. 그래서 어떤 오류든 `catch(Exception e)`로 잡힌다.

정확한 오류 타입(`ArrayIndexOutOfBoundsException`)을 알았으니, 구체적으로 잡고 마지막에 포괄 처리를 둔다.

```java
try {
    arData[10] = 10;
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("인덱스의 범위를 벗어나 값을 대입하였습니다.");
} catch (Exception e) {          // 그 외 모든 예외 (부모는 뒤에!)
    System.out.println(e);
}
```

> ⚠️ **`catch` 순서 주의**: 구체적인 예외를 먼저, 포괄적인 `Exception`을 **나중에** 둬야 한다. 부모 타입을 앞에 두면 자식 예외가 도달하지 못한다.

---

## 📝 정리

```
익명 클래스 & 예외 처리
├─ 익명 클래스  이름 없이 그 자리에서 일회성 구현 (매개변수 자리)
├─ 예외 처리    try-catch-finally로 에러 잡아 흐름 유지
├─ Exception    모든 예외의 부모 타입
└─ catch 순서   구체적 예외 → 포괄(Exception) 순
```

| 개념 | 한 줄 정의 |
|------|------|
| **익명 클래스** | 이름 없이 즉석에서 구현하는 일회용 클래스 |
| **try-catch** | 오류를 잡아 프로그램 중단을 방지 |
| **Exception** | 모든 예외의 최상위 부모 |
| **finally** | 오류와 무관하게 항상 실행 |

익명 클래스는 처음엔 매개변수 안의 긴 코드가 낯설지만, "그 자리에서 일회용으로 구현한다"는 개념만 잡으면 이해된다. 예외 처리는 견고한 프로그램의 필수 요소이니, 구체적 예외부터 잡는 습관을 들이자.
