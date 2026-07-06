---
title: "[C] 구조체"
date: 2023-11-10 14:10:00 +0900
categories: [Programming-Language, C]
tags: [c]
render_with_liquid: false
---

## 📌 들어가며

배열은 **같은 타입**의 데이터만 묶을 수 있었다. 그런데 "학생 한 명"을 표현하려면 학번(문자열), 나이(정수), 키(실수)처럼 **서로 다른 타입**을 하나로 묶어야 한다. 이걸 가능하게 하는 것이 **구조체(struct)**다.

> **구조체(struct)란?**
> 여러 개의 데이터(**멤버**)로 구성된 **사용자 정의 자료형**. 한 구조체에 여러 타입의 변수를 함께 저장할 수 있다.

예를 들어 학생 데이터를 구조체로 묶으면 이렇게 표현된다.

| 멤버 | 타입 |
|------|------|
| 학번 | `char[]` |
| 이름 | `char[]` |
| 나이 | `unsigned char` (0~255) |
| 학년 | `unsigned char` |
| 성별 | `char` ('M'/'F') |
| 키 | `float` (173.4) |

---

## 1. 구조체 정의와 사용

`point`라는 구조체를 만들어 보자. 위치는 `main()` 함수 밖, 함수처럼 둔다.

```c
struct point {
    int x;   // 멤버 변수(member variable)
    int y;
};
```

![Desktop View](/assets/img/Programming-Language/C/Structure/1.png){: width="100%" }

`x`, `y`처럼 구조체 안의 변수들을 **멤버 변수**라고 한다. 이제 `main()`에서 사용해 보자.

```c
struct point p1;   // point 구조체 타입의 변수 p1
p1.x = 100;        // 멤버 연산자(.)로 멤버에 접근
p1.y = 200;
printf("(%d, %d)\n", p1.x, p1.y);
```

![Desktop View](/assets/img/Programming-Language/C/Structure/2.png){: width="100%" }

`p1`의 타입은 `point` 구조체다. 멤버에 값을 대입할 때는 **멤버 연산자(`.`)**를 쓴다. `p1.x = 100;`은 "p1의 멤버 x에 100을 대입"이라는 뜻이다.

![Desktop View](/assets/img/Programming-Language/C/Structure/3.png){: width="100%" }

여러 타입을 섞은 `person` 구조체도 만들어 보자.

![Desktop View](/assets/img/Programming-Language/C/Structure/4.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Structure/5.png){: width="100%" }

> 💡 구조체 멤버에 **문자열을 넣을 때는 `strcpy`**를 쓴다. 이때 소스 파일 맨 위에 `#include <string.h>`를 잊지 말자.

![Desktop View](/assets/img/Programming-Language/C/Structure/6.png){: width="100%" }

**초기화 방법**도 여러 가지다.

![Desktop View](/assets/img/Programming-Language/C/Structure/7.png){: width="100%" }

선언과 동시에 배열 형태로 초기화할 수 있다.

![Desktop View](/assets/img/Programming-Language/C/Structure/8.png){: width="100%" }

`{0}`으로 초기화하면 **모든 멤버가 0**이 된다. `char` 멤버는 널(`'\0'`) 문자로 초기화되어 출력 시 아무것도 나오지 않는다. (배열과 비슷한 부분이다.)

---

## 2. typedef — 타입에 별명 붙이기

`main()` 위에서 정의한 구조체를 본문에서 쓰려면 매번 `struct person p1;`처럼 `struct`를 붙여야 한다. 길어지면 번거롭다. **`typedef`**는 원하는 타입명으로 **별명**을 붙여준다.

```c
typedef struct person PS;   // 이제 struct person 대신 PS로

PS p1;   // struct person p1; 과 동일
```

![Desktop View](/assets/img/Programming-Language/C/Structure/9.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Structure/10.png){: width="100%" }

**구조체 정의와 동시에** typedef를 붙이는 방법도 있다.

```c
typedef struct _person {
    ...
} Person;   // 앞으로 struct _person 을 Person 으로
```

![Desktop View](/assets/img/Programming-Language/C/Structure/11.png){: width="100%" }

---

## 3. 구조체의 크기 (size)

`Person`(char 20 + int + double)의 크기는 얼마일까? 단순 합산이면 `20 + 4 + 8 = 32byte`다.

```c
printf("sizeof(Person) = %d\n", sizeof(Person));
```

![Desktop View](/assets/img/Programming-Language/C/Structure/12.png){: width="100%" }

일단 **32byte**가 출력된다.

> 💡 실제로는 CPU가 데이터를 효율적으로 읽도록 **메모리 정렬(padding)** 때문에 단순 합보다 커질 수 있다. 자세한 내용은 추후 정리.

---

## 4. 구조체에 대한 포인터

구조체에도 포인터를 적용할 수 있다.

```c
Person *ptr = &p3;   // p3의 주소를 ptr에 저장
```

![Desktop View](/assets/img/Programming-Language/C/Structure/13.png){: width="100%" }

자주 쓸 서식 문자열을 전역변수로 만들어두면 편하다.

```c
char *fmt = "이름:%s, 나이:%d, 몸무게:%.1lf\n";
```

![Desktop View](/assets/img/Programming-Language/C/Structure/14.png){: width="100%" }

포인터로 멤버에 접근하는 두 가지 방법을 비교하자.

| 표기 | 의미 |
|------|------|
| `(*ptr).name` | 멤버 연산자(`.`)가 참조 연산자(`*`)보다 우선순위가 높아 **괄호가 필요** |
| `ptr->name` | 위와 **완전히 동일**. C가 만든 간편 표기(화살표 연산자) |

즉 `(*ptr).name`을 더 간단히 쓴 것이 **`ptr->name`**이다. 작동은 똑같다.

![Desktop View](/assets/img/Programming-Language/C/Structure/15.png){: width="100%" }

이를 이용해 **구조체 포인터를 매개변수로 받는 함수**도 만들 수 있다.

```c
void printPerson(Person *ptr) { ... }

printPerson(ptr);    // Person *ptr = &p3; 인 경우
printPerson(&p3);    // 직접 주소를 넘겨도 됨
```

![Desktop View](/assets/img/Programming-Language/C/Structure/16.png){: width="100%" }

---

## 5. 구조체와 배열

구조체를 배열 형태로도 쓸 수 있다.

```c
Person people[3];   // Person 구조체 3개짜리 배열
```

![Desktop View](/assets/img/Programming-Language/C/Structure/17.png){: width="100%" }

단순 배열과 다른 점은, **각 배열 원소 안에 또 여러 멤버 변수**가 들어 있다는 것이다. (`people[0].name`, `people[0].age` …)

![Desktop View](/assets/img/Programming-Language/C/Structure/18.png){: width="100%" }

![Desktop View](/assets/img/Programming-Language/C/Structure/19.png){: width="100%" }

---

## 📝 정리

```
구조체(struct)
├─ 정의     struct 이름 { 멤버들; };   서로 다른 타입을 하나로
├─ 접근     변수.멤버  (멤버 연산자 .)
├─ typedef  struct 이름에 별명 부여 → 간결하게 선언
├─ size     멤버 합 (+ 메모리 정렬 padding)
├─ 포인터   (*ptr).name  ≡  ptr->name  (화살표 연산자)
└─ 배열     Person people[N] → 원소마다 멤버들 보유
```

| 개념 | 한 줄 정의 |
|------|------|
| **구조체** | 서로 다른 타입을 묶는 사용자 정의 자료형 |
| **멤버 연산자 `.`** | 구조체 변수의 멤버에 접근 |
| **typedef** | 타입에 별명을 붙여 선언을 간결하게 |
| **화살표 `->`** | 포인터로 멤버 접근, `(*ptr).x`의 축약 |

구조체는 "서로 다른 데이터를 하나의 의미 단위로 묶는" 도구다. 특히 `ptr->name`(=`(*ptr).name`) 표기는 앞으로 정말 자주 만나게 되니 확실히 익혀두자. 긴 C 여정이었는데 보람이 있었다!
