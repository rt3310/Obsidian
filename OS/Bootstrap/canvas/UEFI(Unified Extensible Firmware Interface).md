![[UEFI.png]]
- 일반적으로 BIOS는 Legacy BIOS를 일컫는데 UEFI는 기존의 Legacy BIOS를 대체할 수 있는 최신 버전의 펌웨어이다.
- Legacy BIOS는 디스크의 첫 섹터에 저장된 MBR을 읽고 부팅과정을 진행하지만 UEFI는 특정 섹터를 지정하지 않고 저장 가능한 GPT 파티션을 사용한다.
- 부팅장치로 MBR이 아닌 GPT를 사용하여 기존의 MBR은 장치의 최대 용량이 2TB였던 것에 비해 GPT는 최대 9ZB까지의 용량을 사용할 수 있다.