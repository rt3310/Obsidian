```yaml
datasource: // [1]
  friend:
    shards: // [2]
      - username
        password
        master:
          name: master-friend
          url
        slaves: // [3]
          - name: slave-friend-1
            url
      - username
        password
        master:
          name: master-friend-2
          url
        slaves:
          - name: slave-friend-1
            url
  ...
```
[1]: DB 접속 정보를 가진 데이터소스를 우선 등록한다.
[2] friend DB의 샤드를 두 개로 나눈 설정이다. shards property에는 샤딩될 DB마다 HA구성으로 설정한다.
[3] slave DB는 여러 대 가질 수 있도록 slave-[moduleName]-[index]형태의 이름을 가지며 RoundRobin으로 밸런싱 해준다.