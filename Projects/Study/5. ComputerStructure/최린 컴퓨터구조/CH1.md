## CPU

발열문제로 인해 전력소모가 최대 150W를 넘어가지 않는다. (보통 100W)
DRAM빼고 CPU칩은 다 NAND 게이트(SRAM도 NAND 게이트).

$T_{exe}$(Execution time per program) = $NI * CPI_{execution} * T_{cycle}$

- NI: 한 프로그램이 실행해야 할 명령어의 수
	- 작을수록 좋다.
- CPI: 명령어 당 필요한 클럭 사이클 수
	- CPI가 작을수록 좋다. 다시 말해, IPC가 높을수록 좋다.
- $T_{cycle}$: 한 클럭 사이클 시간
	- 클럭 사이클 시간이 작을수록 좋다. 다시 말해, 클럭 속도가 높을수록 좋다. 