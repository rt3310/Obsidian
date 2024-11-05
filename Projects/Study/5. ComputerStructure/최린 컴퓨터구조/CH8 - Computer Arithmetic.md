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
### 변환이 쉽다 (비트에 보수를 취한 후 N + 1)
- 3 -> 00000011
- Boolean complement gives -> 11111100
- Add 1 to LSB -> 11111101
### Overflow는 다음과 같은 상황에서만 발생한다
- 두 숫자의 부호 비트가 같디만 연산 결과가 반대 부호를 가질 때
- When the sign bit of two numbers are the same but the result has the opposite sign
### 산술이 쉽다
- A-B를 수행하려면 B에 2의 보수를 취한 후 A에 더한다

## 덧셈과 뺄셈
### 덧셈
- 일반 이진 덧셈
- Monitor sign bit for overflow
### 뺄셈
- 감수에 2의 보수를 취하여 피감수에 더한다
- 따라서 가산기와 보수 회로만 있으면 된다.

## 곱셈
### 원리
- 각 숫자에 대한 부분 곱셈 계산
- 각 부분 곱셈 시프트
- 부분 결과들을 더한다
- Note: 두 배 길이의 결과가 필요하다

## Signed 곱셈
### Unsigned binary multiplication alrogithm
- 부호 있는 곱셈에서는 작동하지 않는다
### 해결 1
- 양수로 바꾼다
- 곱한다
- 부호가 다르면 부정을 취한다.
### 해결 2
- Booth's algorithm

## Unsigned 정수 나눗셈
### Unsigned 이진 나눗셈
- shift와 뺄셈으로 구현할 수 있다.
- 곱셈 하드웨어는 나눗셈에도 사용할 수 있다.

## Signed 나눗셈
### Signed 이진 나눗셈
- 곱셈보다 좀 더 복잡하다

1. Unsigned 이진 나눗셈 알고리즘은 음수로 확장될 수 있다.
2. M에 제수를 로드하고 A, Q에 피제수를 로드한다.
3. A, Q를 1bit 위치만큼 왼쪽으로 이동한다.
4. M과 A가 동일한 부호를 갖는 경우 A <- A - M을 수행한다. 그렇지 않으면 A + M을 수행한다.
- 2~4번을 n번 반복한다.
- A에는 나머지. 제수와 피제수의 부호가 같으면 몫은 Q에, 그렇지 않으면 Q의 2의 보수이다.

## 실수
### Numbers with fractions