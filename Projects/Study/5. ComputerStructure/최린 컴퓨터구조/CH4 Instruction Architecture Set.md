## A Simple Program Translation

m.c(ASCII source file) -[Translator]-> p (Binary executable object file(memory image on disk))

> [!note]
> 문제점:
> - Efficiency: 작은 변경에도 완전한 recompilation이 필요하다.
> - Modularity: 공통 기능들을 공유하기 어렵다 (e.g. printf)
> 
> 해결책: **Separate compilation: use linker**

## A Better Scheme Using a Linker

![[Pasted image 20241028005955.png]]
## Compiler Driver
Compiler driver는 translation과 linking process의 모든 과정을 조정한다. 
- 일반적으로 각 컴파일 시스템에 포함된다. (e.g. gcc)
- preprocessor(cpp). compiler(cc1), assembler(as), linker(ld)를 호출한다.
- command line arguments를 적절한 단계로 전달한다.