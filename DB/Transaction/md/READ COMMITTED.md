Read Committed는 커밋된 데이터만 조회할 수 있다.
Read Committed는 Repeatable Read에서 발생하는 Phantom Read에 더해 **Non-Repeatable Read(반복 읽기 불가능) 문제**까지 발생한다.