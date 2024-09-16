`#pragma`는 **컴파일러에게 말하는 전처리기 명령**이다. 즉, `#include`나 `#define`처럼 전처리기에 의해 컴파일 이전에 처리되지만, 그 명령은 컴파일러에게 전달된다.
사실 `pragma`는 C언어의 기본 키워드라고 하기 보다는, 컴파일러에 종속적인 키워드라고 하는 것이 맞다. `pragma`를 사용하는 문법은 컴파일러마다 다르고 딱히 통일된 것이 없기 때문이다.

```c
#include <stdio.h>
struct Weird {
	char arr[2];
	int i;
};
int main() {
	struct Weird a;
	printf("size of a : %d \n", sizeof(a));
	return 0;
}
```
성공적으로 컴파일 했다면
```
size of a : 8
```
와 같이 나온다.

상당히 이상하다. 분명히 `Weird` 구조체 내의 원소들의 총 바이트 수를 계산해보면 `arr`은 `char`형 변수 2개로 2바이트이고, `i`는 `int`형 변수 1개로 4바이트이므로 6이 나와야 정상이다. 그런데 컴퓨터는 왜 8로 출력했을까?

왜냐하면 실제로 메모리 상에서 위 구조체의 크기를 8바이트로 컴파일러가 지정했기 때문이다. 현재 우리가 사용하는 컴퓨터에서는 언제나 4바이트 단위로 모든 것을 처리하는 것이 빠르다. 따라서 언제나 컴퓨터에서 데이터를 보관할 때는 4의 배수로 데이터를 보관하는 것이 처리 시 용이하게 된다.
이렇게 데이터가 4의 배수 경계에 놓은 것을 **더블 워드 경계에 놓여있다** 라고 한다.

이러한 이유 때문에 위 `Weird` 구조체 역시 4의 배수를 맞추기 위해 크기를 8바이트로 '필요없는 2바이트를 추가하면서 까지' 맞춘 것이다. 이 문제가 중요하게 여겨지는 부분은 역시 하드웨어 간의 통신 때문에 그렇다.

예를 들어서 `SCSI` 인터페이스는 `PC`에서 하드디스크와 같은 주변 기기에 연결하기 위한 통신 방식으로 `SCSI` 장치들에게 읽기 명령을 내리기 위해서는 6바이트의 명령어를 전송하면 된다. 이 6바이트 명령어의 구조는 꽤 복잡해서 흔히 구조체로 많이 이용하는데, 만일 위와 같이 사용했다가는 구조체의 크기가 8바이트로 설정되어서 무슨 문제가 생길지 알 수 없다.

이렇게 컴파일러로 하여금 구조체를 더블 워드 경계에 놓지 말라고 하고 싶을 때 `pragma` 키워드를 이용하면 된다.
```c
#include <stdio.h>
#pragma pack(1)

struct Weird {
	char arr[2];
	int i;
};
int main() {
	struct Weird a;
	printf("size of a : %d \n", sizeof(a));
	return 0;
}
```
성공적으로 컴파일 했다면
```
size of a : 6
```
와 같이 나온다.

이번에는 6으로 잘 나온다. 위 명령은 마이크로소프트 계열의 컴파일러들에만 유효한 문장인데, 구조체를 '1 바이트 단위로 정렬하라는 뜻'이다. 즉, 구조체의 크기가 1의 배수가 되게하라는 것이다. 1 외에도 2, 4, 8, 16 등이 올 수 있다. 만일 기본값, 즉 더블 워드 경계로 정렬하기 위해서는 `#pragma pack(4)` 로 하면 된다.

## `#pragma once`

아까의 `Weird` 구조체 예제에서 `Weird` 부분만 다른 헤더파일로 빼놓아 보자. 이 헤더파일의 이름은 `weird.h`이다.
```c
/* weird.h */
struct Weird {
	char arr[2];
	int i;
}
```

```c
/* test.c */
#include <stdio.h>
#include "weird.h"

int main() {
	struct Weird a;
	a.i = 3;
	printf("Weird 구조체의 a.i : %d \n", a.i);
	return 0;
}
```
성공적으로 컴파일 했다면
```
Weird 구조체의 a.i : 3
```
와 같이 나온다.

`test.c`에서 `weird.h`를 포함했으므로 `weird.h`의 내용이 `test.c`로 그대로 복사된 셈이다. (즉, `#include "weird.h"` 부분이 `weird.h`의 내용으로 바뀌었다고 봐도 무방하다) 따라서 `struct Weird`를 사용할 수 있게 되므로 위와 같은 결과가 발생한다. 그런데 만일 실수로 `weird.h`를 두 번 포함했다고 해보자. 그럼 어떻게 될까?

컴파일 했다면
> [!ERROR] 컴파일 오류
> error C2011: 'Weird' : 'struct' 형식 재정의
> 'Weird' 선언을 참조하십시오.

위와 같이 오류를 만나게 된다. 왜냐하면 각각 `#include "weird.h"` 부분이 `weird.h`의 내용으로 바뀌어서 결과적으로는
```c
#include <stdio.h>
struct Weird {
	char arr[2];
	int i;
};
struct Weird {
	char arr[2];
	int i;
};
int main() {
	struct Weird a;
	a.i = 3;
	printf("Weird 구조체의 a.i : %d \n", a.i);
	return 0;
}
```
를 한 것과 마찬가지가 되어서 `struct Weird`를 두번 정의했다고 오류가 나게 된다. 이를 막으려면 어떻게 해야될까? 일단 C의 기본 전처리기 명령을 이용하여 하는 방법이 있다.
```c
/* 수정된 weird.h */
#ifndef WEIRD_H
#define WEIRD_H
struct Weird {
	char arr[2];
	int i;
};
#endif
```

```c
/* 이상한 test.c */
#include <stdio.h>
#include "weird.h"
int main() {
	struct Weird a;
	a.i = 3;
	printf("Weird 구조체의 a.i : %d \n", a.i);
	return 0;
}
```
성공적으로 컴파일 했다면
```
Weird 구조체의 a.i : 3
```
와 같이 잘 실행된다. 일단 왜 오류가 나지 않는지 살펴보자. 우리가 전처리기라고 한다면 맨 처음에 첫번째 `#include "weird.h"` 를 만났을 때 `WEIRD_H` 가 정의되어 있지 않으므로 `#ifndef` 가 참이 되어 아래 `#define` `WEIRD_H` 가 수행되어 `WEIRD_H` 라는 것이 정의된다. (값은 모르지만 아무튼, 이러한 이름이 정의되었다고 해보자)

또한 헤더파일의 내용도 `test.c` 로 그대로 복사된다. 그 후에 실수로 `weird.h` 를 다시 한 번 `include` 하였을 때 에는 이미 `WEIRD_H` 가 정의되어 있는 상태이므로 `#ifndef WEIRD_H` 가 거짓이 되어 `#endif` 로 넘어가버려 `test.c` 에 그 내용이 복사가 안된다.

이렇게 하면 헤더파일의 내용이 중복으로 포함되는 것을 막을 수 있다. (이는 이미 수많은 헤더파일에서 사용되고 있는 방법이다) 하지만 `#pragma` 를 이용하면 훨씬 단순하게 할 수 있는데,
```c
#pragma once
struct Weird {
	char arr[2];
	int i;
};
```

```c
#include <stdio.h>
#include "weird.h"
int main() {
	struct Weird a;
	a.i = 3;
	printf("Weird 구조체의 a.i : %d \n", a.i);
	return 0;
}
```
성공적으로 컴파일 했다면
```
Weird 구조체의 a.i : 3
```
와 같이 잘 나온다. 이 명령은 컴파일러로 하여금 이 파일이 오직 딱 한 번만 `include` 될 수 있다는 것을 말해주는데, 이는 위에서 `#ifndef` 를 이용하여 복잡하게 하였던 작업들을 단순하게 한 문장으로 끝낼 수 있게 된다.

또한 `#pragma once` 의 장점으로 `#ifndef` 를 이용하는 것 보다 컴파일 시간을 절약할 수 있다는 점인데, `#ifndef` 를 이용하게 되면 `include` 하였을 때 전처리기가 직접 헤더파일을 열어 보아서 과연 `WEIRD_H` 가 정의되었나 정의되지 않았나 확인해 보아야 하는데, `pragma once` 를 이용하면 한 번 `include` 되었다면 헤더파일을 다시 열어보지도 않기 때문에 컴파일 시간이 절약되는 효과가 나게 된다.

다만 앞에서도 말했듯이 `#pragma` 관련 키워드들이 컴파일러 종속적이여서 어떤 컴파일러에서는 `#pragma once` 가 지원이 되지 않을 수도 있다. 따라서 무슨 컴파일러를 사용하는지 보고 `#pragma once` 를 지원한다면 되도록 이것을 사용하는 것이 도움이 된다.

실제로 아래 코드는 [stdio.h](https://modoocode.com/34) 의 헤더파일을 열어본 것이다.
```c
/***
 *stdio.h - definitions/declarations for standard I/O routines
 *
 *       Copyright (c) Microsoft Corporation. All rights reserved.
 *
 *Purpose:
 *       This file defines the structures, values, macros, and functions
 *       used by the level 2 I/O ("standard I/O") routines.
 *       [ANSI/System V]
 *
 *       [Public]
 *
 ****/
#if _MSC_VER > 1000
#pragma once
#endif

#ifndef _INC_STDIO
#define _INC_STDIO

/* 내용 (생략) */

#endif /* _INC_STDIO */
```
위 헤더파일에서 사용하는 컴파일러마다 어떠한 키워드를 사용할 수 있게 하였는지 알 수 있는데,
```c
#if _MSC_VER > 1000
#pragma once
#endif
```
를 보면 `_MSC_VER` 이 1000 보다 크면 `#pragma once` 키워드를 사용하라고 되어있다. `_MSC_VER` 은 마이크로소프트 사의 전처리기에 의해 기본적으로 정의되어 있는 상수로 컴파일러의 버전을 나타내는데, `Visual C++` 의 경우 `_MSC_VER` 값이 1000 부터 시작 하여 현재 2008 버전은 1500 의 값을 가지고 있다. 즉, 현재 버전의 컴파일러의 경우 `_MSV_VER > 1000` 이 참이 되므로 `#pragma once` 키워드를 이용하게 된다. 구 버전의 컴파일러는 그 아래
```
#ifndef _INC_STDIO
#define _INC_STDIO
…
#endif /* _INC_STDIO */
```
과 같이 C 표준 방식의 형태를 사용하도록 되어 있는 것을 볼 수 있다.