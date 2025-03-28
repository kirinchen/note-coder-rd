---
title: JPA Spring Data 常用寶典
tags: [java, jpa, springData]

---

# JPA Spring Data 常用寶典

## @annotation

### @Transient
```java=
    @Transient
    @JsonIgnore
    public static String getUidByInfo(UidArgs args) {
        return null;
    }
```
不會當成欄位值


## 常見問題
### No EntityManager with actual transaction available for current thread - cannot reliably process 'remove' call

會遇到這個問題,就是在一個Transactional有新增又有刪除的情況發生,最簡的的辦法就是在 Dao 加上以下 Annotation

```cmake=
    @Transactional
    @Modifying
```
ex:
```java=
@Repository
public interface RunLogDao extends CrudRepository<RunLog, Long> {

    List<RunLog> findByRunFlowUidOrderByCreateAtAsc(String uid);

    @Transactional
    @Modifying
    List<RunLog> deleteByCreateAtBefore(Date date);
}
```

###### tags: `java` `jpa` `springData`