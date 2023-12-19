---
title: "[Java] 파일입출력(FileIO)"
date: 2023-11-16 00:00:00 +0900
categories: [Programming-Language, Java]
tags: [java]
render_with_liquid: false
future: true
---

이번 시간에는 저번 시간에 배웠던 파일 입출력이어서 쭉 해보겠습니다. 이번 포스팅은 아주 짧아요!

```
package day20;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class FoodFileTest {

	public static void main(String[] args) throws IOException{		
        BufferedWriter bw = new BufferedWriter(new FileWriter("food.txt"));
		bw.write("자장면\n");
		bw.write("짬뽕\n");
		bw.write("소고기\n");
		bw.write("닭고기\n");
		bw.close();
     }
}
```

먼저 라인마다 텍스트를 집어넜었습니다. 이제 이 자료들을 수정하려고 합니다. 수정이 조금 까다로운데 진행해보겠습니다. 기본 로직은 임시 저장소를 만들고 원래 자료를 넣습니다. 그리고 수정할 데이터는 수정을 하고 넣은 뒤, 그 임시 저장소를 원본에 덮어쓰기를 하는 식으로 진행됩니다.

여기서는 닭고기를 찜닭으로 수정을 해보겠습니다.

```
BufferedReader br = new BufferedReader(new FileReader("food.txt"));
		String line = null;
		String temp = "";
		while((line = br.readLine()) !=null) {
			if(line.equals("닭고기")) {
				temp +="찜닭\n";
				continue;
			}
			temp +=line + "\n";
		}
		br.close();
```

먼저 BufferedReader로 우리가 작성했던 텍스트를 불러옵니다. 그 불러온 것은 line에 넣는데요. 아까 말했던 임시 저장소 temp도 만들어줍시다. 여기에는 대입이 아닌 연결을 할거라 null이 아닌 ""로 초기화를 했습니다. 그리고 while 문은 저번에 배웠던 로직 그대로입니다. 추가 된 것은 if문으로 가져온 line이 "닭고기"문자열이면 temp에 우리가 바꿀 "찜닭\\n"을 넣는거죠. 그리고 그 밑으로 내려가서 닭고기도 temp에 들어가면 안되니까 continue로 다음 반복을 걸었습니다. 그럼이제 닭고기는 찜닭으로 변경되고 나머지는 temp+=line + "\\n"으로 변경없이 들어갑니다.

그럼 실행하면 food.txt에는 우리가 수정한 값이 들어가있을까요? 아닙니다. 왜냐하면 우리가 수정한 내용은 그저 임시저장소에 있기 때문이죠. 이것을 다시 덮어써야 합니다.

```
BufferedWriter bw2= new BufferedWriter(new FileWriter("food.txt"));
		bw2.write(temp);
		bw2.close();
```

이러면 되겠죠? 이제 다시 봅시다

![Desktop View](/assets/img/Programming-Language/Java/FileIO/1.png)

정상적으로 수정이 된 것을 볼 수 있습니다. 재밌네요 ^^

그럼 이제 삭제를 해볼까요? 우리는 저 4개중에 소고기가 맘에 들지 않습니다. 소고기를 삭제하겠습니다.

삭제는 수정보다 더 간단합니다. 이제 틀린그림 찾기를 해봅시다.

```
BufferedReader br = new BufferedReader(new FileReader("food.txt"));
		String line = null;
		String temp = "";
		while((line = br.readLine()) !=null) {
			if(line.equals("소고기")) {
				continue;
			}
			temp +=line + "\n";
		}
		br.close();
		
		BufferedWriter bw2= new BufferedWriter(new FileWriter("food.txt"));
		bw2.write(temp);
		bw2.close();
```

무엇이 달라졌을까요? 네 맞습니다. 소고기를 찾은뒤 원래는 temp 에 찜닭을 넣었는데 이번에는 그런 거 없이 그냥 continue로 가버렸습니다. 따라서 소고기를 만나면 그 라인을 temp에 넣지 않고 건너뛰는 것이죠.

![Desktop View](/assets/img/Programming-Language/Java/FileIO/2.png)

정상적으로 삭제가 되는것을 볼 수 있습니다.

이로써 2달간의 JAVA공부가 끝났네요.. 그 시간이 어떻게 지나갔는지도 모르게 새로운 세상을 경험했습니다. 앞으로는 국비교육을 받으면서 배웠던 것들과 느낀 점들을 포스팅 해보려고 합니다!! 그동안 봐주셔서 감사합니다 ^~^