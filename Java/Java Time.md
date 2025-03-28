---
title: Java Time
tags: [java]

---

# Java Time

## 兩時間相減
```java=
        ZonedDateTime k1t = ZonedDateTime.parse("2022-04-28T00:40:00+00:00");
        ZonedDateTime k2t = ZonedDateTime.parse("2022-04-28T00:35:00+00:00");
        double v= ChronoUnit.MILLIS.between( k1t,k2t) / 1000;
        System.out.println(v);

// -300.0
        double v= ChronoUnit.MILLIS.between(k2t, k1t) / 1000;
        System.out.println(v);
// 300.0
```

## +秒數
```java=
   ZonedDateTime date = ZonedDateTime.parse("2017-03-28T12:25:38.492+05:30[Asia/Calcutta]");
      System.out.println(date.plusSeconds(20));  
```

### plus by ChronoUnit
```java
        Instant instant 
            = Instant.parse("2018-12-30T19:34:50.63Z"); 
  
        // add 30 DAYS to Instant 
        Instant value 
            = instant.plus(30, ChronoUnit.DAYS); 
```

### Timestamp to ZonedDateTime,LocalDateTime 
```java=
import java.sql.Timestamp;
import java.time.LocalDateTime;
...
        Timestamp timestamp = new Timestamp(System.currentTimeMillis());
        Instant i= timestamp.toInstant();
        ZonedDateTime ans = instant.atZone(<ZoneId>);
```

### ZonedDateTime to Date
```java!
    Date.from(java.time.ZonedDateTime.now().toInstant());
```

###### tags: `java`