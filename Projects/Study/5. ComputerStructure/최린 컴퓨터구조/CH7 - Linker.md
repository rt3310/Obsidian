## Linking with Static Libraries

### Static library
- 컴파일 타임에, 링커에 입력으로 제공할 수 있는 관련된 object modules package
	- 링커는 프로그램에서 실제로 참조하는 라이브러리의 object modules만 복사한다.
- ex)
	- libc.a: ANSI C standard C library that includes printf, scanf, strcpy, etc.
	- libm.a: ANSI C math library
### Instead of
- unix> gcc main.c /usr/lib/printf.o /usr/lib/scanf.o
### Do
- unix> gcc main.c /usr/lib/libc.a
### To create a library, use AR tool
- unix> gcc -c addvec.c multvec.c
- unix> ar rcs libvector.a addvec.o multvec.o
[gcc로 정적 라이브러리 파일 만들기](https://velog.io/@hidaehyunlee/GCC%EB%A1%9C-%EC%A0%95%EC%A0%81-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%ED%8C%8C%EC%9D%BC-%EB%A7%8C%EB%93%A4%EA%B8%B0)

## Packaging Commonly Used Functions

### 프로그래머가 일반적으로 사용하는 함수를 패키징하는 방법은?
- Math, I/O, memory management, string manipulation, etc.
### 지금까지의 linker framework를 고려할 때 불편한 부분
- Option 1: 모든 함수를 하나의 source file에 넣는다.
	- 프로그래머가 프로그램에 큰 object file을 연결한다.
	- 공간 및 시간 비효율성
		- 각 executable object file은 디스크 공간과 메모리 공간을 차지한다.
		- 한 기능을 변경하려면 전체 source file을 recompilation 해야한다.
- Option 2: 각 함수를 별도의 source file에 넣는다.
	- 프로그래머는 적절한 binary를 프로그램에 명시적으로 연결한다.
	- 더 효율적이지만, 프로그래머에게 부담된다.
### 해결책: static libraries (.a archive files)
- 연관된 relocatable object file을 index(archive라고 한다)를 사용하여 단일 파일로 연결한다.
- 링커가 하나 이상의 archive에서 symbol을 찾아 해결되지 않은 external references를 해결하도록 향상시킨다.
- archive member file이 references를 해결하면, member file을 exectuable file에 연결한다.

## Using Static Libraries

### Linker's algorithm for resolving external references
- o. 파일 및 .a 파일을 command line 순서로 스캔한다.
- 스캔하는 동안 현재 해결되지 않은 참조 목록을 보관한다.
- 각 새로운 .o 또는 .a 파일 obj가 만나면, 목록에서 해결되지 않은 각 참조를 obj의 symbol과 비교하여 해결한다.

As each new .o or .a file obj is encountered, try to resolve each unresolved reference in the list against the symbols in obj.
- If there exist any entries in the unresolved list at end of scan, then error.
### Problem
- Command line order matters
- Moral, put libraries at the end of the command line