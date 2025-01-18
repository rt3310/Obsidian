### FriendConfig
```java
@Configuration
@EnableJpaRepositories( ... )
@ConfigurationProperties(prefix = "datasource")
public class FriendConfig {

    private ShardingDataSourceProperty friend; // [1]

    @Bean
    public DataSource friendDataSource() {
        DataSourceRouter router = new DataSourceRouter(); // [2]
        Map<Object, Object> dataSourceMap = new LinkedHashMap<>(); // [3]

        for (int i = 0; i < property.getShards().size(); i++) {
            ShardingDataSourceProperty.Shard shard = property.getShards().get(i);

            DataSource masterDs = dataSource(shard.getUsername(), shard.getPassword(), shard.getMaster().getUrl());
            dataSourceMap.put(i + SHARD_DELIMITER + shard.getMaster().getName(), masterDs); // [4]

            for (ShardingDataSourceProperty.Property slave : shard.getSlaves()) {
                DataSource slaveDs = dataSource(shard.getUsername(), shard.getPassword(), slave.getUrl());
                dataSourceMap.put(i + SHARD_DELIMITER + slave.getName(), slaveDs);
            }
        }

        router.setTargetDataSources(dataSourceMap); // [5]
        router.afterPropertiesSet(); // [6]

        return new LazyConnectionDataSourceProxy(router); // [7]
    }

    ...
}
```
**[1]** yaml에 설정한 프로퍼티(datasource.[module].shards)를 갖는다.
**[2]** DataSourceRouter는 AbstractRoutingDataSource를 확장하여 만든 클래스이다. 타겟 데이터소스를 등록하고 실제 라우팅을 처리하는 클래스이다. 아래에서 자세히 설명한다.
**[3]** 여러 데이터소스 정보를 담기 위한 map이다.
**[4]** 데이터소스 map의 키는 "샤드 넘버 + delimiter + 샤드 이름" 형태를 가진 lookup key이다.
이 키를 이용해 DataSourceRouter에서 데이터소스를 추출하여 사용한다.
**[5]** 라우터에 데이터소스 map을 등록한다.
**[6]** AbstractRoutingDataSource의 afterPropertiesSet()을 호출한다.
**[7]** LazyConnectionDataSourceProxy는 커넥션의 효율적인 활용 뿐만 아니라 멀티 데이터소스(샤딩 및 MHA 구성에 따른)를 구성하기 위해서 필요한 클래스이다.
데이터소스의 Connection 획득이 실제 쿼리 호출 시에 이루어지도록 함으로써 라우터(아래 DataSourceRouter 참고)가 determineCurrentLookupKey() 메소드를 통해 타겟 데이터소스를 결정할 수 있게 한다.