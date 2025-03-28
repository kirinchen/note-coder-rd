---
title: Java Stream
tags: [java]

---

# Java Stream

## find Index
```java
IntStream.range(0, users.size())
    .filter(userInd-> users.get(userInd).getName().equals(username))
    .findFirst()
    .getAsInt();
```

## collect 指定 型別

```java
List<String> fields=Arrays.asList("foo", "bar", "baz", "foo");
CopyOnWriteArrayList<String> l =
    fields.stream()
          .distinct()
          .collect(Collectors.toCollection(CopyOnWriteArrayList::new));
 ```  
 
## collection to map
```java=
 var map= l.stream().collect(Collectors.toMap(e -> e.positionSide, e-> e));
```

### groupingBy to List
```java=
Map<BlogPostType, List<BlogPost>> postsPerType = posts.stream()
  .collect(groupingBy(BlogPost::getType));
```

進階
```java=
List<CacheKBar> list= cacheKBarDao.findByExchangeAndPair(exPair.exchange,exPair.pair);
        return list.stream()
                .collect(Collectors.groupingBy(
                        CacheKBar::getSer,
                        Collectors.mapping(CacheKBar::load, Collectors.toList())
                ));
```


###### tags: `java`