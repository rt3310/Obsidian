## ELF Symbol Table

```cpp
typedef struct {
	int name;       // string table offset
	int value;      // section offset, or VM address
	int size;       // object size in bytes
	char type:4;    // data, func, section, or src file name(4bits)
		 binding:4; // local or global(4bits)
	char reserved;  // unused
	char section;   // section header index, ABS, UNDEF, or COMMON
} Elf_Symbol;
```

## String and Weak Symbols

### Symbol resolution
- 링커는 각 reference를 정확히 한 symbol 정의와 연결시킨다.
- 동일한 모듈에 정의된 local symbol에 대한 참조는 간단하다.
- 전역 변수에 대한 참조를 해결하는 것이 더 까다롭다.
	- The same symbol might be defined by multiple object files
### Program symbols are either strong or weak
- strong: 프로시저와 초기화된 전역 변수
- weak: 초기화되지 않은 전역 변수
	- At compile time, the compiler exports each global symbol in the assembler as either strong or weak, and the assembler encodes the information implicitly in the symbol table of the relocatable object file
![[Pasted image 20241028124158.png]]