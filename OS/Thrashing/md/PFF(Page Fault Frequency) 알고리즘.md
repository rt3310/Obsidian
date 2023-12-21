Degree Of MultiProgramming을 조절하면서 스레싱을 방지한다는 건 동일하지만, Working Set을 추정하기 보단 페이지 부재율(Page Fault Rate)을 보면서 조절한다는 차이가 있다.

일반적으로 프로그램에게 할당되는 메모리의 크기가 커지면 페이지 부재율(Page Fault Rate)은 줄어든다.
그래서 특정 프로그램에서 페이지 부재(Page Fault)가 많이 발생한다면 메모리를 더 많이 줘서 스레싱을 해겨