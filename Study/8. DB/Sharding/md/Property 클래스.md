### ShardingDataSourceProperty
```java
@Getter
@Setter
public class ShardingDataSourceProperty {
    private List<Shard> shards;

    @Getter
    @Setter
    public static class Shard {
        private String username;
        private String password;
        private Property master;
        private List<Property> slaves;
    }

    @Getter
    @Setter
    public static class Property {
        private String name;
        private String url;
    }
}
```

### ShardingProperty
```java
@Getter
@Setter
public class ShardingProperty {
    private ShardingStrategy strategy;
    private List<ShardingRule> rules;
    private int mod;

    @Getter
    @Setter
    public static class ShardingRule {
        private int shardNo;
        private long rangeMin;
        private long rangeMax;
    }
}
```