---
title: "[Java] 파일입출력 예외처리"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

## 📌 들어가며

이번 글에서는 **스트림(Stream)** 기반의 파일 입출력과 **예외 처리(try-catch, throws)**를 다루고, 마지막으로 **인터넷 이미지 다운로드 앱**까지 만들어본다.

> **스트림(Stream)이란?**
> 강 줄기처럼 흘러가는 것. 컴퓨터에서 그 대상은 **데이터**다. 자바는 `java.io` 패키지에서 스트림 API를 지원한다.

**방향에 따른 분류** (기준은 실행 중인 프로그램):

```
     [입력]                프로그램                [출력]
파일/키보드 ──read──►  [ 실행 중 ]  ──write──►  파일/화면
```

| 클래스 | 단위 | 특징 |
|------|------|------|
| `FileInputStream` | **1 byte** | 가장 근본적, 느림 |
| `FileReader` | byte **묶음** | 유니코드 호환(문자 단위) |
| `BufferedReader` | 줄(`\n`) | 파일 입출력에 가장 많이 쓰임 |

---

## 1. 파일 복사 — FileInputStream

이미지를 1byte씩 읽어(`read`) 빈 파일에 1byte씩 쓰는(`write`) 복사 프로그램이다.

```java
class ImageFileCopy {
    FileInputStream fis;   // 읽기 스트림
    FileOutputStream fos;  // 쓰기 스트림
    public ImageFileCopy() {
        try {
            fis = new FileInputStream(".../gini3.jpeg");   // 원본
            fos = new FileOutputStream(".../gini1.jpeg");  // 빈 파일 생성

            int data = -1;
            while (true) {
                data = fis.read();      // 1byte 읽기
                if (data == -1) break;  // 파일 끝(-1)이면 종료
                fos.write(data);        // 1byte 쓰기
            }
        } catch (FileNotFoundException e) {
            System.out.println("해당 파일을 찾을 수 없습니다.");
        } catch (IOException e) {
            System.out.println("파일 읽기에 실패했습니다.");
        } finally {
            if (fis != null) try { fis.close(); } catch (IOException e) {}
            if (fos != null) try { fos.close(); } catch (IOException e) {}
        }
    }
    public static void main(String[] args) { new ImageFileCopy(); }
}
```

> 💡 `read()`는 데이터가 없으면 **-1**을 반환한다. 그래서 `data == -1`이면 반복을 종료한다.

![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/1.png)

---

## 2. 예외 처리 (Exception Handling)

> **문법에 문제가 없어도 프로그램이 무사히 수행된다는 보장은 없다.** 외부 요인(파일 없음, 네트워크 끊김 등)으로 **비정상 종료**될 수 있는데, 이는 컴퓨터 분야에서 가장 무서워하는 상황이다. 이를 막는 기술이 **예외 처리**다.

```
정상 흐름 ──예외 발생──► [원래는 비정상 종료]
                              │ try-catch가 가로챔
                              ▼
                        catch로 진입 (e로 원인 전달) → 로그·피드백 → 안정화
```

> 💡 **예외 처리의 목적 = 비정상 종료 방지.** 예외가 발생하면 예외 객체가 `e`로 전달되고, 개발자는 이를 이용해 원인 분석·로그 기록을 한다. 원상 복구는 불가능하므로 주로 **로그를 남기거나 관리자에게 피드백**한다.

**`finally`**는 (switch의 default처럼) 예외 발생 여부와 무관하게 **항상 실행**된다. 파일 입출력은 `close()`로 마무리해야 하는데, 이를 finally에 둔다.

> ⚠️ `close()` 전에 `if (fis != null)`로 검사하는 이유: 경로가 잘못돼 `fis`가 null이면 `close()` 호출 시 `NullPointerException`이 나기 때문. **null이 아닐 때만** 닫는다.

---

## 3. 문자 단위 복사 — FileReader (+ throws)

> `FileInputStream`의 `read()`는 **1byte**만 읽어 한글(유니코드 2byte)을 표현할 수 없다. 영문은 1byte라 문제없지만, 한글은 깨진다. 해결책은 **2byte씩 묶어 문자로 이해하는 `FileReader`**다.

```java
class ReadCharacter {
    FileReader reader;
    FileWriter writer;
    // throws: 예외 처리를 호출부(JVM)에게 '전가'
    public ReadCharacter() throws FileNotFoundException, IOException {
        reader = new FileReader(".../memo.txt");
        writer = new FileWriter(".../memo_copy.txt");
        int data = -1;
        while (true) {
            data = reader.read();
            if (data == -1) break;
            writer.write(data);
        }
        if (reader != null) reader.close();
        if (writer != null) writer.close();
    }
    public static void main(String[] args) throws IOException {
        new ReadCharacter();
    }
}
```

| 방식 | 설명 |
|------|------|
| **try-catch** | 각 예외에 대해 개발자가 **반응을 정의** |
| **throws** | 예외 처리를 **호출부에 던져버림**(전가) |

---

## 4. 표준 입출력 — System

> 무심코 쓰던 `System.out.println`도 사실 입출력이다. `System` 클래스에는 **`err`, `out`, `in`** 세 개가 있다. `out`은 표준 출력, `in`은 표준 입력(키보드)이다.

```java
InputStream is = System.in;   // 표준 입력(키보드)
int data = -1;
while (true) {
    data = is.read();               // 키보드로부터 1byte 읽기
    System.out.print((char) data);
}
```

---

## 5. 응용 — 인터넷 이미지 다운로드

URL로부터 스트림을 만들어, 원격 이미지를 로컬에 저장하는 앱이다. (길지만 핵심은 `getImage()` 하나다.)

```java
public void getImage() {
    try {
        url = new URL(t_url.getText());              // 입력한 주소로 URL 생성
        URLConnection con = url.openConnection();
        httpCon = (HttpURLConnection) con;           // 다운캐스팅
        is = httpCon.getInputStream();               // 원격 자원 → 스트림

        String ext = FileManager.getExt(t_url.getText());   // 확장자 추출
        fos = new FileOutputStream(".../ginigini" + ext);   // 저장 파일

        int data = -1;
        while (true) {
            data = is.read();
            if (data == -1) break;
            fos.write(data);                         // 로컬에 복사
        }
        JOptionPane.showMessageDialog(this, "읽기 완료");
    } catch (MalformedURLException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (is != null)  try { is.close(); }  catch (IOException e) {}
        if (fos != null) try { fos.close(); } catch (IOException e) {}
    }
}
```

핵심 흐름은 **"원격 URL → 스트림 → 로컬 파일"**이다.

```
URL(주소) → URLConnection → HttpURLConnection → getInputStream() → is
                                                                     │
로컬 파일 ◄── FileOutputStream.write() ◄── read() ◄────────────────┘
```

> 💡 로컬 파일 복사와 원리는 **완전히 같다.** 차이는 입력 스트림이 로컬 파일이 아니라 **네트워크(HttpURLConnection)**라는 것뿐이다.

![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/2.png)
![Desktop View](/assets/img/Programming-Language/Java/FileIO-Exception/3.png)

---

## 📝 정리

```
파일 입출력 & 예외 처리
├─ 스트림    FileInputStream(1byte) / FileReader(문자) / Buffered
├─ 복사      read()로 읽고 write()로 씀, -1이면 끝
├─ 예외      try-catch(반응 정의) / throws(전가) / finally(항상)
├─ 표준 I/O  System.in / out / err
└─ 응용      URL → 스트림 → 파일 (네트워크 복사)
```

| 개념 | 한 줄 정의 |
|------|------|
| **스트림** | 데이터가 흘러가는 통로 |
| **try-catch vs throws** | 반응 정의 vs 호출부에 전가 |
| **finally** | 예외 여부와 무관하게 항상 실행(close) |
| **HttpURLConnection** | 네트워크 자원을 스트림으로 |

파일 입출력의 핵심은 **"읽어서(read) 쓴다(write), 끝나면 닫는다(close)"**이다. 그리고 로컬이든 네트워크든 스트림으로 추상화하면 **동일한 코드**로 다룰 수 있다는 점이 스트림의 강력함이다.
