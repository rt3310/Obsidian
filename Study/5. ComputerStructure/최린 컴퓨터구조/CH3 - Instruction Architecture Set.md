## MIPS Addressing Modes
### Register addressing
- Address is in a register
- jr $ra (indirect branches)
### Base addressing
- Address는 register(base register) content 와 상수(offset)의 합이다.
- ldw \$s0, 100(\$s1)
### Immediate addressing
- 상수 operand의 경우 (immediate operand)
- add $t1, $t2, 3
### PC-relative addressing
- Address는 PC 와 상수(offset)의 합이다.
- beq $s0, $s1, L1
### Pseudodirect addressing
- Address는 PC의 상위 비트와 연결된 26비트 offset이다.
- j L1

## Procedure call & return
### Steps of procedure call & return
- **callee**가 접근할 수 있는 공간에 파라미터들을 넣는다.
	- $a0 ~ $a3: four **argument registers**
- callee로 제어권을 넘긴다.
	- ja1 callee_address: **jump and link** instrunction
		- \$ra에 return address(PC+4)를 넣고 callee로 jump한다.
- callee에게 필요한 저장 공간(**stack frame**)을 얻는다.
- 원하는 작업을 수행한다.
- **caller**가 접근할 수 있는 공간에 결과값을 넣는다.
	- $v0 ~ $v1: two value registers to return values
- caller에게 제어권을 반환한다.
	- jr $ra

> [!note] procedure vs function
> - procedure: return 값이 있는 것
> - function: return 값이 없는 것

## Stack
### Stack frame(aka. activation record) of a procedure
- procedure에 local로 변수를 저장한다.
	- Procedure에 저장된 레지스터들
		- Arguments, return address, saved registers, local variables
- Stack pointer ($sp): stack의 top을 가리킨다.
- Frame pointer ($fp): 상위 stack frame의 bottom을 가리킨다.