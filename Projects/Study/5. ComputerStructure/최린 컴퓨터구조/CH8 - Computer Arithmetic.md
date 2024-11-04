## Arithmetic & Logic Unit
### Roles of ALU
- Does the (integer arithmetic) computations
- Everything else in the computer is there to service this unit
### FPU (floating point unit)
- Arithmetic unit that handles floating point (real) numbers
- Operations on separate floating point registers
### Implementation
- All microprocessors has integer ALUs
- Most of recent microprocessors including mobile chips now has on-chip FPU

## 정수형 표현
### Only have 0 & 1 to represent everything
### Two representative representations
- Sign-magnitude
- Two's compliment
### Sign-magnitude
- Left most bit is sign bit
	- 0 means positive
	- 1 means negative
- Problems
	- Need to consider both sign and magnitude in arithmetic
	- Two representations of zero (+0 and -0)

## 2의 보수
### Given N, 2's complement of N with n bits
- $2^n - N = (2^n - 1) - N + 1 =  bit \space complement \space of \space N + 1$
- 32 bit number
	- Positive numbers : 0(x00000000) to $2^{31} - 1$ (x7FFFFFFF)
	- Negative numbers : -1(xFFFFFFFF) to $-2^{31}$ (x80000000)
- Like sign-magnitude, MSB represents the sign bit

## 2의 보수의 특성
### 0 단일 표현
### Negation is fairly easy (bit complement of N + 1)
- 3 -> 00000011
- Boolean complement gives -> 11111100
- Add 1 to LSB -> 11111101
### Overflow occurs only
- When the sign bit of two numbers are the same but the result has the opposite sign
### Arithmetic works easily
- To perform A - B take the 2's complement of B and add it to A
- 