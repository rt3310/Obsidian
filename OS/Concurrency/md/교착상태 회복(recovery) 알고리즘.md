1. 교착상태에 연관된 모든 프로세스 종료 → 많은 운영체제에서 채용
2. 교착상태에 연관된 프로세스들을 체크포인트 시점으로 롤백한 후 다시 수행 시킴 → 교착 상태가 다시 발생할 가능성 존재
3. 교착상태가 없어질 때까지 교착상태에 포함되어 있는 프로세스들 하나씩 종료 시키면서 교착상태 발견 알고리즘을 재실행
4. 교착상태가 없어질 때까지 교착상태에 포함되어 있는 자원을 하나씩 선점(비용이 작은 순 서로) 시키면서 교착상태 발견 알고리즘을 재실행 -> 롤백 필요
	1. 비용이 낮으면 해당 프로세스는 기아상태 가능→비용요소 + 복귀의 횟수
	    
### 3. 종료, 4. 선점 방식에서 프로세스 선택 기준  
- 지금까지 사용한 처리기 시간이 적은 프로세스부터 √ 지금까지 생산한 출력량이 적은 프로세스부터  
- 이후 남은 수행시간이 가장 긴 프로세스부터  
- 할당 받은 자원이 가장 적은 프로세스부터  
- 우선 순위가 낮은 프로세스부터