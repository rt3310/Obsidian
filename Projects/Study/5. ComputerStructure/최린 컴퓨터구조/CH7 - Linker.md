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
- Option 1: 모든 함수를 single source file에 넣는다.
	- 프로그래머가 프로그램에 큰 object file을 연결한다.
	- 공간 및 시간 비효율성
		- 각 executable object file은 디스크 공간과 메모리 공간을 차지한다.
		- Each executable object file occupies disk space as well as memory space
		- Any change in one function would require the recompilation of the retire source file
- Option 2: Put each function in a separate source file
	- Programmers explicitly link appropriate binaries into their programs
	- More efficient, but burdensome on the programmer
### Solution: static libraries (.a archive files)
- Concatenate related relocatable object files into a single file with an O (called an archive)
- Enhance linker so that it tries to resolve unresolved external references by looking for the symbols in one or more archives
- If an archive member file resolves reference, link the member file into executable.