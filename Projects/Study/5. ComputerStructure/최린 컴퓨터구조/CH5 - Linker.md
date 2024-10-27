## Linker
### Linking
- 다양한 코드와 데이터를 수집하여, 메모리에 로드(복사)하고 실행할 수 있는 하나의 파일로 결합하는 프로세스이다.
- 링킹 시간
	- compile time에 수행할 수 있다. (i.e. source code가 translate될 때)
	- 또는, load time에도 (i.e. 프로그램이 메모리로 load될 때)
	- 또는, runtime에도.
### Static Linker
- compile time에 링킹을 수행한다.
- relocatable object files의 collection과 command line arguments를 가져와서 로드하고 실행할 수 있는 fully linked executable object file을 생성한다.
- 2개의 main tasks를 수행한다.
	- Symbol resolution: 각각의 symbol reference를 정확히 한 symbol definition과 연결한다.
	- Relocation: code와 data sections을 재배치하고 재배치된 메모리 위치에 대한 symbol references를 수정한다.
### Dynamic Linker
load time 또는 runtime에 링킹을 수행한다.

## Object files
### Relocatable object file
- executable object file을 만들기 위해 컴파일 시 다른 relocatable object files과 결합할 수 있는 형태의 바이너리 코드와 데이터를 포함한다.
	- 컴파일러와 어셈블러가 relocatable object files을 생성한다.
### Executable object file
- 메모리로 바로 복사되고 실행될 수 있는 형태의 바이너리 코드와 데이터를 포함한다.
	- 링커가 executable object files을 생성한다.
### Shared object file
- 로드 타임이나 런타임에 동적으로 메모리에 로드되고 연결될 수 있는 특별한 유형 relocatable object file
## Relocation Sections

![[Pasted image 20241026212451.png]]

## What Dose a Linker Do?
### Merges object files
- 여러 relocatable object file을 로더에서 로드하고 실행할 수 있는 single executable object file로 병합한다.
### Resolves external references
- 병합 프로세스의 일부로, 외부 참조를 해결한다.
	- 외부 참조: 다른 object file에 정의된 symbol reference
### Relocates symbols
- .o 파일에서의 relatives locations에서 실행가능한 파일에서의 새로운 absolute position(virtual address space)로 symbol을 재배치한다.
- 이러한 symbol에 대한 모든 references를 업데이트하여 새로운 위치를 반영한다.
- References는 code나 data중 하나일 수 있다.
	- code: `a();`
	- data: `int *xp = &x;`

## Why Linkers
### Modularity
- 프로그램은 하나의 monolithic mass가 아닌 더 작은 source files의 모음으로 작성할 수 있다.
- 일반적인 기능의 라이브러리를 구축할 수 있다
	- e.g. math library, standard C library
### Efficiency
- 시간
	- source file하나를 변경하고, 컴파일한 다음 다시 연결하기
	- 다른 source files을 다시 컴파일할 필요가 없다.
- 공간
	- 공통 기능 라이브러리를 single file로 통합할 수 있다.
	- 그러나 실행 파일과 실행 중인 메모리 이미지에는 실제로 사용가능한 코드만 포함되어 있다.

## Object File Format
### Object file format은 시스템마다 다르다
- Examples: a.out(early UNIX systems), COFF(Common Object File Format, used by early versions of system V), PE(Portable Executable, a variation of COFF used by Microsoft Windows), ELF(Used by modern UNIX including Linux)
### ELF(Executable and Linkable Format)
- Standard binary format for object files
- Derives from AT&T System V Unix
	- Later adopted by BSD Unix variants and Linux
- One unified format for
	- Relocatable object files (.o).
	- Executable object files
	- Shared object files (.so)
- Generic name: ELF binaries
- better support for shared libraries than old a.out formats

## ELF Object File Format
### ELF Header
- Word 크기
- 시스템의 byte ordering
- machine 타입(e.g. IA32)
- ELF header의 크기
- object file 타입(e.g. relocatable, executable, or shared).
- section header table의 file offset과 엔트리 크기, 엔트리 수.
### Segment header table
- Page 크기
- memory segments 가상 주소(sections)
- segment 크기
- 접근 권한(readable, writable, ...)
### Section header table
- 다양한 sections의 위치와 크기를 포함한다.
- 각 섹션에 대한 고정 크기 항목
### .text section
- machine code
### .data section
- 초기화된 전역 변수들
- 로컬 C 변수는 스택에 런타임에 유지되며 .data나 .bss section에 나타나지 않는다.
### .bss section
- 초기화되지 않은 전역 변수들
- "Block Storage Start" or **"Better Save Space!"**
- place holder일 뿐 space를 차지하지 않는다.
### .symtab section
- Symbol table
- 함수 및 전역 변수에 대한 정보
### .rel .text section
- .text section의 재배치 정보
- 실행 파일에서 수정해야 하는 명령어의 주소들
### .rel .data section
- .data section의 재배치 정보
- 통합된 실행 파일에서 수정해야하는 pointer data의 주소
### .debug section
- symbolic debugging에 대한 정보 (gcc -g)
### .line section
- source code의 line numbers와 .text section의 machine instructions 간 매핑
### .strtab section
- String table for symbols in the symtab and debug

## Life and Scope of an Object
### Life vs scope
- 객체의 수명은 객체가 (프로세스의) 메모리에 있는지 여부를 결정하는 반면, 객체의 범위는 객체에 접근할 수 있는 코드 범위를 결정한다.
- 객체가 살아있지만 보이지 않을 수 있다.
- 객체가 살아있지 않으면 보일 수 없다.
### Local variables
- 함수 내부에 정의된 변수
- 변수의 범위는 함수 안에만(function scope) 있다.
- 함수가 완료되면 변수의 수명이 끝난다.
- 따라서 함수를 다시 호출하면 변수에 대한 저장소가 생성되고 값이 다시 초기화 된다.
- 정적 로컬 변수 - 값이 프로그램 수명 기간 동안 어느 정도 유지되기를 원한다면 로컬 변수를 "static"으로 정의할 수 있다.
	- 초기화는 첫 번째 호출에서만 수행되며 함수 호출 간에 데이터가 유지된다.
### Global variables
- 함수 외부에 정의된 변수
- 변수의 범위는 전체 프로그램에 걸쳐 있다(global scope).
- 프로그램이 완료되면 변수의 수명이 끝난다.
### Static variables
- 'static' 키워드를 사용하여 선언된 변수: 두 가지 유형
	- 변수는 메모리에 정적으로(영구적으로) 할당된다.
- **정적 변수**: 정적 변수가 전역 공간에 정의된 경우(e.g. 파일 시작 부분), 이 변수는 이 파일에서만 접근할 수 있다(file scope).
	- 정적 변수는 정의된 모듈의 범위가 local이지만 수명은 프로그램 전체에 걸쳐있다.
	- 전역 변수가 있고 파일을 라이브러리로 배포하는 경우, 다른 사용자가 전역 변수에 접근하지 못하도록 하고 싶다면, 'static' 키워드를 접두사로 붙여 정적으로 만들면 된다.
- **정적 로컬 변수**: 함수 내부에 선언된 정적 변수

## Symbols
### Symbol
- 변수나 함수(code)에 대한 참조
	- 함수 이름, 전역 변수, 정적 변수, 정적 로컬 변수
- Linker symbol != program variable
### 세 가지 종류의 linker symbols
- 모듈 m에 의해 정의되고 다른 모듈에서 참조할 수 있는 **global symbols**
	- Nonstatic C 함수, nonstatic 전역 변수
		- C static attribute 없이 정의된 함수 또는 전역 변수
- 모듈 m에서 참조하지만 다른 모듈에서 정의하는 **global symbols**
	- externals라고 부른다.
- 모듈 m에서 독점적으로 정의하고 참조하는 **local symbols**
	- 정적 C 함수,정적 변수
	- 정적 로컬 변수는 스택에서 관리되지 않는다. 대신, 컴파일러는 **.data나 .bss에 공간을 할당**한다.
	- Local linker symbol != local program variable