ZonedDateTime 클래스는 타임존 또는 시차 개념이 필요한 날짜와 시간 정보를 나타내기 위해서 사용된다.
```java
ZonedDateTime nowSeoul = ZonedDateTime.now();
ZonedDateTime nowLa = ZonedDateTime.now(ZoneId.of("America/Los_Angeles"));
ZonedDateTime worldCup = ZonedDateTime.of(2002, 6, 18, 20, 30, 0, 0, ZoneId.of("Asia/Seoul"));
```

-> ZonedDateTime = LocalDateTime + 타임존/시차

## ZoneId vs ZoneOffset
- `ZoneId`는 타임존, `ZoneOffset`은 시차를 나타낸다.
- `ZoneOffset`은 UTC 기준으로 고정된 시간 차이를 양수나 음수로 나타내는 반면에 `ZoneId`는 이 시간 차이를 타임존 코드로 나타낸다.
```java
ZoneOffset seoulZoneOffset = ZoneOffset.of("+09:00");
System.out.println("+0900 Time = " + ZonedDateTime.now(seoulZoneOffset));
ZoneId seoulZoneId = ZoneId.of("Asia/Seoul");
System.out.println("Seoul Time = " + ZonedDateTime.now(seoulZoneId));
```