---
title: "[Java] 배열"
date: 2023-11-12 14:10:00 +0900
categories: [Programming-Language, Java]
tags: [java]
---


이번시간에는 배열에 대해서 알아보겠습니다~

**\*배열 : 저장공간의 나열**

1\. 여러 개의 변수를 선언하면 변수가 많아지기 때문에 관리하기 어렵습니다. 따라서 배열은 여러 칸을 선언하며, 이름은 하나이기 때문에 데이터 관리가 편합니다.

2\. 규칙성이 없는 값에 규칙성을 부여하기 위해서 사용합니다.

**\*배열의 선언**

자료형\[\] 배열명 = {값1, 값2, 값3, ...};

자료형\[\] 배열명 = new 자료형\[칸수\];

자료형\[\] 배열명;

배열명 = new 자료형\[칸수\];

C언어에서는 보통 자료형 배열명 \[\] 으로 하는데 JAVA에서는 자료형 바로 뒤에 대괄호를 붙이네요~

그리고 또 다른 점은 new라는 연산자인데요. 아직 자세히 배우지는 못했지만 이것도 하나의 연산자이며 주 내용은 Heap메모리에 저장시켜준다고 합니다. 그렇다면 선언의 방식 첫번째에서 new없이 중괄호만 쓴다면 heap메모리에 저장이 안 되나요? 아닙니다. 중괄호는 new생략한 것뿐입니다.

따라서 배열은 모두 RAM안에 Heap메모리 안에 들어가게 됩니다. 그래서 자바에서 배열을 선언하면 그것은 모두 동적 배열이겠네요 ^^

**\*배열의 길이(length)**

배열 선언 시 배열의 길이가 length라는 상수에 담깁니다.

배열명.length로 사용합니다.

```
int[] arData = {3,4,6,7};
		
		System.out.println(arData);
		System.out.println(arData[0]);
		System.out.println(arData[arData.length -1]);
		System.out.println(arData.length);

		System.out.println("============");

		for(int i=0;i<arData.length;i++) {
			System.out.println(arData[i]);
		}
```

배열을 하나 선언하고 여러가지를 출력해봤습니다. 결과물을 한번 볼까요?

![Desktop View](/assets/img/Programming-Language/Java/Array/1.png)

오잉? 첫번째는 이상한 게 출력되었네요. 이것은 arData를 출력했을 때 나오는 값인데요. 이것은 arData\[0\], 즉 첫번째 값의 주소값을 나타냅니다.

두번째는 배열의 첫번째값이 출력되었구요, 그 다음은 arData\[arData.length-1\]의 값을 출력한 것인데요. 위에 설명 드린 것처럼 배열의 길이는 length에 상수로 저장이 됩니다. 따라서 arData.length는 4칸이니까 4가 나오겠죠? 여기에 -1을 했으니 3이겠네요. 그렇다면 어떤 배열이든지 length -1 은 그 배열의 마지막 칸을 의미 합니다. 역시 그 다음에 length를 출력했을 때 4가 정상적으로 출력되었습니다.

그 다음엔 for문을 이용하여 arData배열을 출력했는데요, 그 동안 우리가 for문을 배우면서 i값을 0으로 자주 시작했었죠? 이게 배열과 관계가 있습니다. 배열의 첫번칸의 번호는 0입니다. 따라서 for문에서 저렇게 출력을하면 손쉽게 배열을 출력할 수 있습니다~

**\*index가 0부터 시작하는 이유**

배열은 시작 주소를 가지고 있으며, 다음 칸으로 이동할 때마다 시작 주소에 +1씩 더해줍니다.

첫번째 방에 접근하기 위해서는 시작주소 + 0이고, 이를 \[\]로 치환하면 더한 0이 인덱스 번호가 됩니다.

따라서 첫번째 방의 인덱스 번호는 0입니다.

이번엔 출력뿐만 아니라 입력도 해봅시다.

```
		int[] arData = new int[5];
		
		for(int i=0;i<arData.length;i++) {
			arData[i] = 5-i;
		}
		for(int i=0;i<arData.length;i++) {
			System.out.println(arData[i]);
		}
	
```

정수타입의 5칸 배열을 선언한 다음에 for문으로 5-i를 차례대로 입력했습니다. 그렇다면 i는 0부터니까 5,4,3,2,1 이렇게 들어갔겠네요. 그 다음엔 배열을 출력했습니다.

![Desktop View](/assets/img/Programming-Language/Java/Array/2.png)

정상적으로 출력되었습니다!

이제는 여러가지 실습을 해볼게요.

```
		// 100~1까지 100칸 배열에 넣고 총 합 구하기
		
		int[] arData = new int[100];
		int sum=0;
		for(int i=0;i<arData.length;i++) {
			arData[i]=100-i;
			sum += arData[i];
		}
		System.out.println(sum);
	
```

먼저100칸짜리 정수타입의 배열을 선언하고 sum이라는 변수를 하나 만들고 0으로 초기값을 줍니다.

그리고 배열에 100-i를 입력합니다~ i는 0부터 배열의 길이만큼 커집니다. 이렇게 하면 배열에 100부터 1까지 값이 들어가겠지요.

그리고 아까 선언 한 sum을 누적연산자를 통해서 배열의 값을 누적시켜줍니다. 그리고 마지막에 sum을 출력하면 되겠네요~

```
int[] arData = new int[5];
      Scanner sc = new Scanner(System.in);
      int max = 0;
      int min = 0;
      
      for (int i = 0; i < arData.length; i++) {
         System.out.print(i + 1 + "번째 정수 : ");
         //사용자에게 5개의 정수 입력받기
         arData[i] = sc.nextInt();
      }
      
      //max에 첫번째 정수 담기
      max = arData[0];
      //min에 첫번째 정수 담기
      min = arData[0];
      
      for (int i = 1; i < arData.length; i++) {
         //만약 max에 있는 값과 배열의 다음 값을 비교했을 때
         
         //max에 있는 값보다 더 큰 값이 있다면 max에 그 값을 담아준다.
         if(max < arData[i]) {max = arData[i];}
         //min에 있는 값보다 더 작은 값이 있다면 min에 그 값을 담아준다.
         if(min > arData[i]) {min = arData[i];}
      }
      
      System.out.println("최대값 : " + max);
      System.out.println("최소값 : " + min);
```

이번에는 최대값과 최소값을 출력해보겠습니다~

5칸의 배열을 선언하고 max와 min이라는 변수를 만들고 0으로 초기화합니다. 그리고 for문으로 정수값을 입력받도록 합니다. 그리고 배열의 첫번째 값을 일단 max,min에 넣습니다. 그리고 for문과 if문으로 배열에 min과 max보다 커지면 갱신하는 식으로 만들었습니다. 중요한건 for문은 1부터 시작하는데요, 그 이유는 첫번째 값은 이미 min과 max값에 들어가 있기 때문에 그다음인 인덱스 1부터 비교를 해야하기 때문입니다~

마지막은 좀 어려운 문제를 풀어볼게요!

```
//정수를 한글로 변경
      //입력 예 : 1024
      //출력 예 : 일공이사

String hangle = "공일이삼사오육칠팔구";
      Scanner sc = new Scanner(System.in);
      int num = 0;
      String result = "";
      
      System.out.print("정수 입력 : ");
      num = sc.nextInt();
      
      while(num != 0) {
         result = hangle.charAt(num % 10) + result;
         num /= 10;  
      }
      System.out.println(result);
```

정수를 입력하면 한글로 바꾸는 건데요.. 진짜 어려웠습니다 ㅎㅎ

처음엔 문자열 hangle을 만들고 나중에 바꿔줄 한글을 공~구 까지 입력했습니다.

그리고 num이라는 정수타입 변수 선언하고 0으로 초기화 한 뒤, result라는 또 다른 문자열을 만들고 초기화했습니다.

그리고 num에 정수를 입력받고 이번에는 while문을 썼는데요, 배열명.charAt()는 무엇이냐하면, 보통 정수타입 등의 배열은 그냥 배열명\[인덱스\]로 그 값을 표현하는데요 문자열같은 경우는 클래스다 보니 조금 다른 것을 씁니다 그게 charAt()입니다~ 그리고 여기에 num%10을 넣은 이유는 예를 들어 1024를 10으로 나눈 나머지는 4입니다. 그리고 charAt(4)는 "사" 겠죠? 그리고 result에 그 값을 넣습니다. 근데 그냥 넣기만 하는게 아니라 뒤에 +result를 넣은이유는 출력순서때문입니다. 1024를 넣으면 일공이사가 나와야하는데 +result를 하지 않고 그냥 누적연산자를 하면 사이공일로 출력되기 때문입니다. 그리고 num/10을 합니다. 정수에 나누기를 하면 소수점은 버립니다. 따라서 1024에서 102로 가겠죠? 그러면 또 10으로 나눈 나머지는 2가 되겠습니다. 그러면 또 그 인덱스 값을 찾아서 result에 넣습니다. 이렇게 10으로 나누다보면 num은 언젠간 0이 되겠죠? 그 때 while 문을 빠져나옵니다. 그리고 result가 출력됩니다 ㅎㅎ 재밌네요!

![Desktop View](/assets/img/Programming-Language/Java/Array/3.png)

잘 출력이 됐습니다~

이번시간에는 배열에 대해서 알아봤습니다!