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
- Word size, byte ordering of the system, the machine type(e.g. IA32)
- The size of ELF header, the object file type(e.g. relocatable, executable, or shared). the file offset of the section header table and the size and number of entries in the section header table.
### Segment header table
- Page size, virtual addresses of memory segments(sections), segment sizes, access permissions(readable, writable, ...)
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
- Life of an object determines whether the object is in memory (of the process) whereas the scope of an object determines the section of code where the object can be accessed
- It is possible that an object is live but not visible
- It is not possible that an object is visible but not live.
### Local variables
- Variables defined inside a function
- The scope of these variables is only within this function (function scope)
- The life of these variables ends when this function completes
- So when we call the function again, the storage for variables is created and values are reinitialized
- static local variables - If we want the value to be extent throughout the life of a program, we can define the local variable as "static."
	- Initialization is performed only at the first call and data is retained between function calls
### Global variables
- Variables defined outside a function
- The scope of these variables is throughout the entire program (global scope)
- The life of these variables ends when the program completes
### Static variables
- Variables declared using 'static' keyword two types
	- These variables are allocated statically (permanently) in memory
- static variables: if a static variable is defined in a global space (asy at beginning of file) then this variable will be accessible only in this file(file scope)
	- Static variables are local in scope to their module in which they are defined, but life is throughout the program.
	- If you have a global variable and you are distributing your files as a library and you want others not to access your global variable. you may make it static by just prefixing keyword static
	- Static local variables: static variables declared inside a function

## Symbols
### Symbol
- Reference to variable or to a function(code)
	- Function names, global variables, static variables, static local variables
- Linker symbol != program variable
### Three kinds of linker symbols
- Global symbols that are defined by module m and that can be referenced by other module
	- These are called externals
- Local symbols that are defined and referenced exclusively by module m
	- Static C functions and static variables
	- Static local variables are not managed on the stack. Instead, the compiler allocates space in .data or in .bss
	- Local linker symbol != local program variable