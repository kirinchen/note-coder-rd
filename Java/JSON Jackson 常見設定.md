# JSON Jackson 

## 遇到 Unrecognized field, not marked as ignorable

在目標 class 增加 
```
@JsonIgnoreProperties(ignoreUnknown = true)
``` 
此 annotaion


## 遇到泛型要轉物件
```java=
    public static <K, V> Map<K, V> load(String json) throws IOException {
        return objectMapper.readValue(json, new TypeReference<Map<K, V>>() {});
    }
```

###### tags: `java`