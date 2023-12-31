---
title: "[Java] Prologue"
date: 2023-11-11 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---

짧았던 C언어의 포스팅이 끝나고 이제는 JAVA포스팅을 하려고 합니다. 쉬운 길은 결코 아니겠지만 그래서 그만큼 더 재미있고 무언가를 얻어갈 수 있는 시간이 될거라 믿습니다!

**JAVA**

썬 마이크로 시스템즈에서 1995년에 개발한 객체 지향 프로그래밍 언어이며 창시자는 제임스 고슬링입니다. 2010년에 오라클이 썬 마이크로시스템즈를 인수하면서 Java의 저작권을 소유하였다고 하네요. 현재는 OpenJDK는 GPL2(? 뭔진모르겠습니다.)나 오라클이 배포하는 Oracle JDK는 상업라이선스로 2019년 1월부터 유료화정책을 강화하고 있습니다. Java EE는 이클립스 재단의 소유이구요.

C#과 문법적 성향이 굉장히 비슷하다고 하네요.

기존의 C에 객체지향 기능을 추가하다 보니 언어의 사용에 있어 저수준과 고수준의 개념이 충돌하는 부분이 많았던 C++과는 다르게 아예 처음부터 객체지향 언어로 개발되었습니다. 하지만 원시타입은 객체로 취급하지 않기 때문에 모든 것을 객체로 취급하는 언어, 즉 순수 객체지향언어인 Python,Ruby 등과는 살짝 다른 점이 있을수 있겠네요.

Java의 가장 큰 특징은 플랫폼에 독립적인 언어라는 점입니다.소스 코드를 기계어로 직접 컴파일하여 링크하는 C/C++의 컴파일러와 달리 자바 컴파일러는 바이트코드인 클래스 파일(.class)을 생성하고, 이 파일의 바이트코드를 읽은 뒤 기계어로 바꾸어 실행하는 것은 Java Virtual Machine(JVM)입니다.

자바는 3가지로 구성되어 있는데요

JVM(Java Virtual Machine)

Java 프로그램을 실행해줍니다.

JRE(Java Runtime Environment)

JVM을 실행할 때 필요한 라이브러리 파일들과 기타 파일들을 가지고 있습니다.

JDK(Java Development Kit)

JVM과 JRE에 의해 실행되고

JRE외에 개발에 필요한 도구들을 가지고 있습니다.

저는 JDK8 버전을 다운받아서 Java를 다 설치했습니다. 그리고 이를 이용하기 위해 IDE(통합개발환경)을 설치해주었는데요 저는 그 중에 eclipse photon버전으로 다운받았습니다. 이것은 하나의 창에서 여러 프로젝트를 관리할 수 있어서 좋다고 하네요!

**자바의 기본구조**

프로젝트 -> 패키지 -> 클래스 -> 메소드 -> 소스코드

![1](https://github.com/easydong02/TIL/assets/82931413/e14a46e7-99ce-4a57-8eda-0ab2ed07317e)

실행시키고 간단한 것을 했는데요, day01 프로젝트에 day01 패키지를 만들고 PrintTest라는 클래스에 main메소드가 보이시는 것을 확인할 수 있습니다. 그리고 그 블럭안에 System.out.println("신동희"); 를 넣어서 제 이름을 출력해보도록 하겠습니다. 실행은 C언어와는 다르게 컨트롤+f11입니다~

![2](https://github.com/easydong02/TIL/assets/82931413/20d6f08c-7e82-4874-b3e1-02243d3f90e5)

틀린그림찾기 ㅎㅎ; 첫 사진에 하단에 콘솔블럭에 아무것도 없었는데 지금은 콘솔에 신동희라는 것이 적혀있죠?

이번시간은 여기까지 하겠습니다. 앞으로 배울 내용이 많지만 엄청 재미있겠네요 ^^
