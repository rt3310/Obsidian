리눅스 namespace는 **프로세스를 실행할 때 시스템의 리소스를 분리해서 실행할 수 있도록 도와주는 기능**이다.
한 시스템의 프로세스들은 기본적으로 시스템의 리소스들을 공유해서 실행된다. 이를 단일 namespace라고 생각해볼 수 있다. 실제로 리눅스에서는 1번 프로세스(init)에 할당되어 있는 namespace들을 자식 프로세스들이 모두 공유해서 사용하는 구조로 이루어져있다.

현재 리눅스에서 지원하는 namespace는 크게 보면 다음과 같다.
- cgroup(cgroup)
- IPC(ipc)
- network(network)
- mount(mnt)
- PID(pid)
- UTS(user)
- User(uts)
- Time(time)
리눅스에서는 프로세스를 실행할 때 각 네임스페이스 별로 분리해서 실행하는 것이 가능하지만 기본적으로는 1번 프로세스의 네임스페이스를 공유해서 실행된다.