---
title: Springboot mongodb 建立 index
tags: [java, springboot, mongodb]

---

# Springboot mongodb 建立 index


## application.properties

```javascript=
spring.data.mongodb.auto-index-creation=true
```
>如果沒加 就算你的 @annotation 有加也不會增加 index...


## 查詢 index

```javascript=
db.run_log.getIndexes()
```


## 可以建立超過多久就刪除的 index

```java=
    @Indexed(name = "create_at_index", direction = IndexDirection.DESCENDING,expireAfterSeconds=3600)
    private Date createAt;
```

###### tags: `java` `springboot` `mongodb`