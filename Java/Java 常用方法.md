---
title: Java 常用方法
tags: [java]

---

# Java 常用方法

## time

### Iso8601 to ZonedDateTime

```java=
java.time.ZonedDateTime.parse("2015-08-18T00:00+01:00");
```

### get UTC+0 now Date
```java=
Date.from(java.time.ZonedDateTime.now(ZoneId.of("UTC+0")).toInstant());
```

### 兩個時間相隔多久

```java=
    private double subtract(String k1,String k2) {
        var k1t = ZonedDateTime.parse(k1);
        var k2t = ZonedDateTime.parse(k2);
        return ChronoUnit.MILLIS.between(k1t,k2t) / 1000;
    }
```

### before
```java=
        ZonedDateTime st = ZonedDateTime.parse("2015-08-18T00:00+01:00");
        ZonedDateTime ed = ZonedDateTime.parse("2019-08-18T00:00+01:00");
        System.out.println(ed.isBefore(st));
        System.out.println(ed.isAfter(st));
        System.out.println(st.isBefore(ed));
        System.out.println(st.isAfter(ed));
```

ans :
```shell=
false
true
true
false
```

## List / Array

### reverse 排序顛倒
```java=
public static void reverse(List<?> list){ ... }
//
Collections.reverse(arrlst);
```

### Array to List
```java=
Integer[] spam = new Integer[] { 1, 2, 3 };
List<Integer> list = Arrays.asList(spam);
```

### Collection to Array
```java=
// Where x is the collection:
Foo[] foos = x.toArray(new Foo[x.size()]);
```

## stream

### sum numbers
```java=
List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
Integer sum = integers.stream()
  .reduce(0, (a, b) -> a + b);
```

### collect 指定 型別

```java
List<String> fields=Arrays.asList("foo", "bar", "baz", "foo");
CopyOnWriteArrayList<String> l =
    fields.stream()
          .distinct()
          .collect(Collectors.toCollection(CopyOnWriteArrayList::new));
```

## JSON

### dto 忽略 field
```java=
public class MyDto {

    private String stringValue;
    @JsonIgnore
    private int intValue;
}
```

## String

### placehoder 

#### MessageFormat.format
```java=
 String streamName = MessageFormat.format("{0}@kline_{1}", symbol, interval);
```

###### tags: `java`