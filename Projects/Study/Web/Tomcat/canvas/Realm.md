하나의 엔진을 위한 렐름(Realm)은 **사용자 인증과 인가를 담당**한다.
애플리케이션 설정 중에, 관리자는 리소스에 접근이 허용된 롤을 정의하고 렐름이 이러한 정책을 실행하는데 사용된다.
렐름은 텍스트 파일, 데이터베이스 테이블, LDAP 서버 등을 통해 인증할 수 있다.

하나의 렐름은 전체 엔진 또는 top-level 컨테이너에 적용되기에, 한 컨테이너 안의 애플리케이션들은 인증을 위한 리소스를 공유한다.