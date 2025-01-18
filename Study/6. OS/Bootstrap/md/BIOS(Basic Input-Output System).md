BIOS는 컴퓨터의 하드웨어 장치들의 I/O를 관리하는 펌웨어(Firmware)이다.
![[BIOS.png]]
- 컴퓨터에 전원이 공급되면 ROM에 저장되어 있던 BIOS가 가장 먼저 실행된다.
- 컴퓨터의 하드웨어 장치들(H/W) <-> BIOS(Firmware) <-> 운영체제(S/W)
- 컴퓨터의 하드웨어 장치들과 운영체제(OS) 사이에 위치하여 운영체제가 하드웨어의 I/O를 제어할 때 BIOS를 통해 제어하게 된다.
- 최근 PC들은 기존의 Legacy BIOS를 대체하는 **UEFI**가 탑재됐다.

![[ROM.png]]
- 위 사진에서 두 개의 ROM이 있는데 하나는 주로 사용되는 M_BIOS(Main BIOS)가 저장되어 있고, 나머지 하나는 Main BIOS가 고장났을 때를 대비하기 위해 백업 용도의 B_BIOS(Backup BIOS)가 저장되어 있다.