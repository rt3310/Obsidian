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
- Contains binary code and data in a form that can be combined with other relocatable object files at compile time to create an executable object file
	- Compilers and assemblers generate relocatable object files
### Executable object file
- Contains binary code and data in a form that can be copied directly into memory and executed
	- Linkers generate executable object files
### Shared object file
- A special type of relocatable object file that can be loaded into memory and linked dynamically, either at load time or at runtime

## Relocation Sections

![[Pasted image 20241026212451.png]]

## What Dose a Linker Do?
### Merges object files
- Merge multiple relocatable(.o) object files into a single executable object file that can loaded and executed by the loader
### Resolves external references
- As part of the merging process. resolves external references
	- External reference: reference to a symbol defined in another object file
### Relocates symbols
- relocates symbols from their relatives locations in the .o files to new absolute positions (virtual address space) in the executable
- Updates all references to these symbols to reflect their new positions
- References can be in either code or data
	- code: `a();`
	- data: `int *xp = &x;`

## Why Linkers
### Modularity
- Program can be written as a collection of smaller source files, rather than one monolithic mass
- Can build libraries of common functions (more on this later)
	- e.g. math library, standard C library
### Efficiency
- Time
	- Change one source file, compile and then relink
	- No need to recompile other source files
- Space
	- Libraries of common functions can be aggregated into a single file
	- Yet executable files and running memory images contain only code for the functions they actually use

## Object File Format
### Object file format varies from system to system
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
- Contains the locations and sizes of various sections
- A fixed size entry for each section
### .text section
machine code
### .data section
- initialized global variables
- Local C variables are maintained at runtime on the stack and do not appear in .data or .bss section
### .bss section
- uninitialized global variables
- "Block Storage Start" or **"Better Save Space!"**
- Just a place holder and do not occupy space
### .symtab section
- Symbol table
- Information about function and global variables
### .rel .text section
- Relocation info for .text section
- Addresses of instructions that will need to be modified in the executable
### .rel .data section
- Relocation info for .data section
- Address of pointer data that will need to be modified in the merged executable
### .debug section
- info for symbolic debugging (gcc -g)
### .line section
- Mapping between line numbers in source code and machine instructions in the .text section
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