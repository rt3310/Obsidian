![[Cache Stampede 현상.png]]
대규모 트래픽 환경에서 **TTL 값이 너무 작게 설정되면 cache stampede 현상이 발생**할 가능성이 있다.

Look Aside 패턴에서 redis에 데이터가 없다는 응답을 받은 서버가 직접 DB로 데이터를 요청한 뒤, 다시 redis에 저장하는 과정을 거친다.

그런데 위 상황에서 key가 만료되는 순간 많은 서버에서 이 key를 같이 보고 있었다면 모든 애플리케이션 서버에서 DB로 가서 찾게 되는 duplicate read가 발생한다.

또 읽어온 값을 각 redis에 쓰는 duplicate write도 발생되어, 처리량도 다 같이 느려질 뿐 아니라 불필요한 작업이 굉장히 늘어나 요청량 폭주로 인해 장애로 이어질 가능성도 있다.
