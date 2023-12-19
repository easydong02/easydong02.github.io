---
title: "[Java] 수동 컴파일, 수동 빌드 실행"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

이클립스를 쓰지 않아서 예를 들어 다른 패키지에 있는 클래스를 임포트하려면 수동으로 다 제어해주어야 합니다. 오늘은 에딧플러스로 패키지 구축과 그 패키지 안에 있는 다른 클래스를 임포트하는 것까지 해보겠습니다.

일단 저는 src폴더와 bin 폴더를 구분하여 생성해서 java파일은 src폴더에, class파일은 bin폴더에 넣으려고 합니다. 물론 컴파일을 하면 자동으로 말이죠. 그 전에 수동으로 하면서 매커니즘을 한번 알아봅시다.

[##_Image|kage@b8RwYY/btryMmILaCP/IEYUV1JovhfSZCvCwdxf10/img.png|CDM|1.3|{"originWidth":169,"originHeight":151,"style":"alignCenter","width":169}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/1.png)

이게 제 경로입니다.D드라이브에서 bigData폴더에 workspaceOfJava2가 보이네요. 거기에 testapp이라는 폴더를 만들고 bin과 src를 만들었습니다.

[##_Image|kage@EO6UZ/btryJYB4BrU/8ZOEfxACN69dC3267NoX1K/img.png|CDM|1.3|{"originWidth":342,"originHeight":340,"style":"alignCenter","width":342}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/2.png)

[##_Image|kage@pIeIb/btryJXQHkg9/791qWXUf3vmnmdeHF3b6k0/img.png|CDM|1.3|{"originWidth":936,"originHeight":523,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/3.png)

파일 탐색기에서 내 PC를 우클릭하면 저런 메뉴가 나옵니다. 속성을 누른 뒤 나오는 설정에서 쭉 내리시면 '고급 시스템 설정' 누릅니다. 그러면 이런 창이 나옵니다.

[##_Image|kage@cxlr5B/btryMnVaexb/pX3SPnFy7Zwo0hmf6XWfB0/img.png|CDM|1.3|{"originWidth":546,"originHeight":667,"style":"alignCenter","width":546}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/4.png)

[##_Image|kage@ZtcQo/btryMZ7Bych/yKb60KqATSSaahqub5TNA1/img.png|CDM|1.3|{"originWidth":707,"originHeight":738,"style":"alignCenter","width":707}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/5.png)

그리고 시스템 속성에서 고급탭에서 쭉 내리면 환경 변수라는게 보입니다. 처음에 이클립스 설치할 때 java home을 등록했던 그 환경변수가 맞습니다.

그리고 시스템 변수탭에서 새로만들기를 합니다.

[##_Image|kage@b3hVTL/btryM1dheUN/ctRlhth9kyHYAxAFC7AKtK/img.png|CDM|1.3|{"originWidth":745,"originHeight":206,"style":"alignCenter","width":745}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/6.png)

그러면 이런 창이 나오는데요 변수이름엔 classpath, 변수 값엔 아까 testapp폴더의 bin로 지정했습니다. class 파일을 생성하는 경로를 지정하는 것이죠.

이제 문서를 2개 작성해봅시다.

[##_Image|kage@dFVHVK/btryLzO5UzA/5KQDGWm5RE0R6LKD0pEWT0/img.png|CDM|1.3|{"originWidth":172,"originHeight":292,"style":"alignCenter","width":172}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/7.png)

[##_Image|kage@6X1qS/btryKugrf76/eVssIBmKTl6MJ8fYMdK4KK/img.png|CDM|1.3|{"originWidth":936,"originHeight":477,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/8.png)

pen이라는 java파일을 만들었습니다. 메인메서드는 안 만들었습니다. 그건 다른 패키지(폴더)에 구축하겠습니다.

[##_Image|kage@dasews/btryLzO5UyL/fpcNJZGX1tKEtHUKwWV6C1/img.png|CDM|1.3|{"originWidth":171,"originHeight":286,"style":"alignCenter","width":171}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/9.png)

[##_Image|kage@bwg7Ym/btryFLXDP7w/fW3NRFeecaqHMHgk60fH01/img.png|CDM|1.3|{"originWidth":936,"originHeight":471,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/10.png)

이게 아까 만든 Pen클래스를 사용할 java파일입니다. 같은 koreait 패키지안에 있는 usetool이라는 패키지에 있는 UsePen파일입니다. 그런데 패키지랑 import 경로를 보면 com이란 패키지부터 되어있습니다. 왜냐면 전체 경로를 쓸 필요가 없는게 아까 환경 변수로 bin폴더까지는 classpath로 지정했기 때문이죠. 이제 두 java 파일을 class파일로 컴파일 해봅시다. cmd를 킵니다.

[##_Image|kage@0yNA6/btryMmWiiv0/tfNup0kFLVvq3b2BHGNl9k/img.png|CDM|1.3|{"originWidth":936,"originHeight":465,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/11.png)

cmd창을 캡쳐했습니다. 설명해보자면 먼저 Pen.java파일을 컴파일할거라 cd 명령어로 기본 경로를 tool폴더까지 해줍니다(여기에 Pen.java있으니까). 그리고 원래는 javac Pen.java로 하는데 여기엔 -d 뒤에 경로가 있네요. 이것은 같은 폴더(패키지)에 class파일을 만드는 것을 방지하고 지정된 경로로 class파일을 만들게 하기 위함입니다. 근데 왜 bin까지 적었냐면 나머진 코드를 짤 때 com.koreait.tool;로 그 뒤의 경로는 java파일 안에 있기 때문입니다. 그리고 Pen.java를 하고 엔터를 하면 컴파일이 됩니다. 확인해볼까요?

[##_Image|kage@8Igcw/btryLAgah6Y/QBHi4AAgnLuu7CjVcp3Nn0/img.png|CDM|1.3|{"originWidth":936,"originHeight":648,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/12.png)

com폴더는 없었는데 경로가 알아서 폴더까지 생성해주고 안에 class파일을 만든 것을 볼 수 있습니다. 근데 이 클래스파일을 실행시킬 수 있을까요? 없습니다. 왜냐하면 메인메서드는 실제로 이 클래스를 쓸 다른 파일에 있기 때문이죠. 그리고 그것이 UsePen입니다. 아까와 같은 방법으로 UsePen.java파일도 컴파일해주었습니다. 이제 이것을 실행해봅시다.

[##_Image|kage@ck8LKy/btryI5PqvSW/4RQGaTDVSsa4koHibJF9MK/img.png|CDM|1.3|{"originWidth":936,"originHeight":464,"style":"alignCenter","width":886}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/13.png)

실행할 때는 com.koreait.usetool.UsePen 으로 하면 됩니다. 실행할 때는 확장자없이 파일 이름으로만 했죠? 그래서 그런겁니다.잘 실행이 되는 것을 볼 수 있습니다. 그러면 다른 클래스를 임포트할 때 항상 이렇게 번거롭게 해야할까요? 놀랍게도 에딧플러스에서 단축키로 할 수 있습니다. 물론 지정 경로는 다 일일이 해주어야 하지만..

[##_Image|kage@tqbow/btryI5BWHWS/WL21PKJPkB5YGw0A6IyVYK/img.png|CDM|1.3|{"originWidth":636,"originHeight":556,"style":"alignCenter","width":636}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/14.png)

[##_Image|kage@0Yx0P/btryM0McmA5/e1LykckWyDOCu3MRCKnh7k/img.png|CDM|1.3|{"originWidth":636,"originHeight":556,"style":"alignCenter","width":636}_##]
![Desktop View](/assets/img/Programming-Language/Java/Compile-Build/15.png)

왼쪽은 컴파일, 오른쪽은 실행입니다.

도구 - 사용자 도구 구성에 들어가면 저렇게 나와있습니다. 일단 아무 그룹이나 잡아서 그룹이름을 편의성을 위해 프로젝트이름과 같이 했습니다. 그리고 컴파일과 실행 두가지 명령을 만들었습니다. 우리가 cmd에서 했던것과 같습니다. 명령에는 실행할 파일을 선택합니다. 컴파일엔 javac가 쓰이고 실행은 java.exe가 쓰이겠죠? 그래서 이 컴퓨터에 설치되어있는 경로를 찾아서 선택합니다.그리고 인수가 중요합니다.

cmd에서 javac -d D:\\bigData\\workspaceOfJava2\\testapp\\bin 파일이름.java로 했었죠?

그래서 인수에

\-d D:\\bigData\\workspaceOfJava2\\testapp\\bin 하고 오른쪽에 화살표를 누르면(파일이름)이 보입니다. 이렇게 하면 각 파일이름대로 cmd에 파일이름.java를 입력하는겁니다. 그러면 ctrl+1하면 그게 나옵니다.

실행을 보겠습니다. 명령에는 java.exe가 있는 폴더에서 선택합니다. 그리고 인수는 아까 cmd 할 때

java com.koreait.usetool.파일이름 으로 했었죠? 같습니다. 그런데 com.koreait.usetool은 경로가 아닌 코드에 짜여진거라 그거까지 스마트하게 확인해주진 못합니다 따라서 똑같이 화살표를 누르면 여러가지 선택지중에 인수묻기가 있습니다. 이걸 누르면 $(Prompt).이게 나옵니다. 이러면 실행할때 프롬프트가 나와서 우리가 com.koreait.usetool와같이 적어주면 됩니다. 그리고 실행을 할 때는 확장자 없는 파일이름을 적으니까 화살표를 눌러서 확장자를 뺀 파일이름을 누르면 $(FileNameNoExt)가 나옵니다. 그럼 ctrl +1로 컴파일을 하고 ctrl+2로 실행하면 됩니다.

오늘은 좀 하면서 자바의 컴파일과 실행에 대해서 깊이 배울 수 있어서 너무 좋았습니다 ^^