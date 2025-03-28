---
title: 'Java Record vs final field class 使用lombok 比較 '
tags: [lombok, java, java17+]

---

在我們常用不可修改DTO常用這種寫法(例如打包的參數)
```java=
    public static class FC1{
        public final ZonedDateTime start;
        public final String me;

        public FC1(ZonedDateTime start, String me) {
            this.start = start;
            this.me = me;
        }
    }
```
現在新版多了個簡易的寫法如下
```java=
    public  record R1(ZonedDateTime start, String me){    }
```
新舊版本使用如下
```java=
    public static void main(String[] args) {
        var fc1 =new FC1(ZonedDateTime.now(),"sss");
        System.out.println("fc1 start="+fc1.start+" me="+fc1.me);
        var r1 = new R1(ZonedDateTime.now(),"sss");
        System.out.println("r1 start="+r1.start+" me="+r1.me);
    }
// fc1 start=2023-07-04T15:23:23.562880400+08:00[Asia/Taipei] me=sss
// r1 start=2023-07-04T15:23:23.573739900+08:00[Asia/Taipei] me=sss
```

# 使用 lombok @Builder
在舊版時我們常用 @Builder 搭配 final field 的做法,現在在Record也可以使用
```java=
 @Builder
    public static class FC1{
        public final ZonedDateTime start;
        public final String me;
    }

    @Builder
    public  record R1(ZonedDateTime start, String me){    }

    public static void main(String[] args) {
        var fc1 =FC1.builder().start(ZonedDateTime.now()).me("BBB").build();
        System.out.println("fc1 start="+fc1.start+" me="+fc1.me);
        var r1 = R1.builder().start(ZonedDateTime.now()).me("BBB").build();
        System.out.println("r1 start="+r1.start+" me="+r1.me);
    }
```

# 序列化的部分
在final field 在序列化通常都要做特殊處理才能用或是放棄final,Record則無這限制
```java=
    public static void main(String[] args) {
        String json =QuickMap.genSimple().add("start",99).add("me","zzz").toJson();
        var r1 =      JsonUtils.loadByJson(json,R1.class);
        System.out.println("r1 start="+r1.start+" me="+r1.me);
        var fc1 =JsonUtils.loadByJson(json,FC1.class);
        System.out.println("fc1 start="+fc1.start+" me="+fc1.me);
    }
```
line:5 執行會throw exception
```shell!
Exception in thread "main" net.bzk.infrastructure.ex.BzkRuntimeException: com.fasterxml.jackson.databind.exc.InvalidDefinitionException: Cannot construct instance of `net.bzk.infrastructure.TestRecord$FC1` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (String)"{"start":99,"me":"zzz"}"; line: 1, column: 2]
```
在Jackson這裡需改成
```java=
    public static class FC1 {
        public final int start;
        public final String me;

        @Builder
        public FC1(@JsonProperty("start") int start, @JsonProperty("me") String me) {
            this.start = start;
            this.me = me;
        }
    }
```
# @EqualsAndHashCode
上述幾乎都是 Record 可以取代原本 final field 的寫法,現在來說 Record 的限制, lombok @EqualsAndHashCode 可以直接產生 defaut field Equals 和 HashCode ,但是無法使用在 Record

```java=
    @EqualsAndHashCode
    public static class FC1 {
        ...
            
      
    @Builder
    @EqualsAndHashCode
    public static record R1(int start, String me) {
        ...
    // throw error: @EqualsAndHashCode is only supported on a class.
    @EqualsAndHashCode        

```

# 結論
在無需使用 custom Equals 和 HashCode methdo,基本上可以用 Record 來替代 final field class 的寫法