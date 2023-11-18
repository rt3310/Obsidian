1. 자바 프로그램을 실행하면 JVM은 OS로부터 메모리를 할당받는다.
2. 자바 컴파일러(javac)가 자바 소스코드(.java)를 자바 바이트 코드(.class)로 컴파일 한다.
3. **Class Loader**는 동적 로딩을 통해 필요한 클래스들을 로딩 및 링크하여 **Runtime Data Area(실질적인 메모리를 할당받아 관리하는 영역)**에 올린다.
4. **Runtime Data Area**에 로딩 된 바이트 코드는 **Execution Engine**을 통해 해석된다.
5. 이 과정에서 **Execution Engine**에 의해 **Garbage Collector**의 작동과 **Thread 동기화**가 이루어진다.