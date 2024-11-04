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

## Integer Representation
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