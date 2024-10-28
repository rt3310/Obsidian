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
	- 동일한 symbol이 여러 object file에 의해 정의될 수 있다.
### Program symbols are either strong or weak
- strong: 프로시저와 초기화된 전역 변수 -> **중복되면 안된다**.
- weak: 초기화되지 않은 전역 변수
	- 컴파일 시에, 컴파일러는 각 global symbol을 어셈블러에 strong이 weak로 내보내고, 어셈블러는 relocatable object file의 symbol table에 정보를 암시적으로 인코딩한다.
![[Pasted image 20241028124158.png]]

## Linker's Symbol Resolution Rules
### Rule 1. string symbol은 하나만 존재해야 한다
### Rule 2. weak symbold은 같은 이름의 strong symbol에 의해 override될 수 있다.
- References to the weak symbol resolve to the strong symbol
### Rule 3. 여러 weak symbol이 있다면, linker는 마음대로 하나를 선택할 수 있다.

## Relocation

### symbol resolution 단계 이후
- symbol resolution 단계가 완료되면, 링커는 코드의 각 symbol reference를 정확히 하나의 symbol definition과 연결한다.
- 링커는 각 object module의 코드와 데이터 섹션의 정확한 크기를 알고있다.
### 재배치는 2단계로 구성되어 있다.
- sections 및 symbol definition 재배치
	- 동일한 타입의 모든 section을 새 aggregate section으로 병합한다.
		- 모든 input module의 data section이 single .data section으로 병합된다.
	- 새로운 aggregate section과 각 모듈에 내부적으로 정의된 각 symbol에 runtime 메모리 주소를 할당한다.
- section 내에서 symbol reference 재배치
	- 외부 참조가 올바른 runtime address를 가리킬 수 있도록 수정
		- 이 단계를 수행하려면 
		- To performs this step, the linker OO on relocation entries to the relocatable object modules