서버는 톰캣 그 자체로 **웹 애플리케이션 서버의 인스턴스**이며 top-level 컴포넌트이다.
서버는 서버를 종료하기 위한 한 개의 포트를 가지며, 디버그 모드 설정을 통해 JVM 디버깅을 시작할 수 있다.

하나의 머신에 애플리케이션을 분리하여 각각 재시작할 수 있도록 각각 서버들을 설정할 수도 있다. 또한, 하나의 JVM에서 실행되는 한 서버에서 오류가 발생하더라도 다른 서버에는 영향을 주지 않고 분리되어 운영될 수 있다.