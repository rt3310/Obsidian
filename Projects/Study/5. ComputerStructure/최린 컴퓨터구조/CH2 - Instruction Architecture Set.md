## Compiler



## Machine State
### Register
메모리로부터 fetch한 Data를 저장하는 CPU 내부 스토리지
- single cycle에 읽고 쓸 수 있다.
- 연산은 레지스터에 저장된 데이터에 의해 수행될 수 있다.
- MINS ISA: 32개 register = 32개 플립플롭
- register <-> caches <-> memory <-> hard disk
	- register는 프로그래머한테 visible
	- cache는 프로그래머한테 invisible
### Memory
0번부터 시작하는 linear array of a byte
- 32비트 register 컴퓨터의 메모리 최대 공간 = $2^{32}$ bytes = 4GB -> 프로그램의 최대 크기 4GB
- 프로그램 저장 (코드, 데이터)

## Data Size & Alignment
### Data Size
- Word: 연산의 기본 단위
	- 32bit for 32bit ISA, 64bit for 64bit ISA
- Double word: 64bit
- Half word: 16bit
### Byte addressability
- 가상메모리에서 모든 byte마다 주소를 가진다. (초창기에는 word 단위로 address하기도 했다)
### Alignment
- 메모리에 저장된 데이터/명령어의 시작 주소가 데이터/명령어 크기의 배수에 저장되게 되어있다.
- byte: 0, 1, 2, 3, 4, 5, 6, 7, 8, ...
- Half word: 0, 2, 4, 6, 8, ...
- Word: 0, 4, 8, ...
- Doubleword: 0, 8, ...

## Machine Instruction
### Opcode : specifies the operation to be performed
- ADD, MULT, LOAD, STORE, JUMP
### Operands
- Source operands (input data)
- Destination operands (output data)
- The location can be
	- Memory operand specified by a memory address
		- ex) $(R2), s1004
	- Register operand specified by a register member
		- R1

## Instruction Types
### Arithmetic and logic instructions
### Data transfer instructions (memory instructions)
- 메모리에서 레지스터로 데이터 이동(LOAD) 또는 반대(STORE)
- Input/Output 명령어들은 보통 메모리 명령들로 구현된다(memory-mapped IO)
	- 메모리 주소 공간에 매핑된 IO devices -> PORT
### Control transfer instructions (branch instructions)
- 프로그램 제어 흐름을 변경한다.
	- fetch할 다음 명의 주소를 결정한다.
		- Program Counter(PC): 다음 실행할 명령어의 주소를 갖고 있는 특수한 레지스터 
- Unconditional jumps and conditional branches
- Direct branches and indirect branches
- ex) JUMP, CALL, RETURN, BEQ

## MIPS Instruction Format

### R-type
| 6   | 5   | 5   | 5   | 5     | 6     |
| --- | --- | --- | --- | ----- | ----- |
| op  | rs  | rt  | rd  | shamt | funct |
> op는 모두 0이고 fucnt을 opcode로 사용
- op: Opcode, 명령어의 기초 연산
- rs: 1st source register
- rt: 2nd source register
- rd: destination register
- shamt: shift amount (immediate operand)
- funct: function code, the specific variant of the opcode
- 산술 논리 명령에 사용된다.
### I-type
| 6   | 5   | 5   | 16      |
| --- | --- | --- | ------- |
| op  | rs  | rt  | address |
> 모두 0을 제외한 나머지 63개를 opcode로 사용
- 데이터 이동, 조건 분기, immediate
- rs: base register
- rt: destination register for load and source register for store
- address: +/-2^32 bytes offset (aka. displacement) -> rs(register)에 메모리의 base 주소가 들어있고 거기에 address(offset)을 더한다
- 저장 및 불러오기, 조건 분기에 사용된다.
### J-type
| 6   | 26      |
| --- | ------- |
| op  | address |
- 명령어 jump