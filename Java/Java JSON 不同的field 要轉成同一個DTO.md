---
title: Java JSON 不同的field 要轉成同一個DTO
tags: [java, json, jackson]

---

在用 jackson JSON 常有種情境就是不同的來源JSON要轉成同一個DTO物件 如下
```json=
{
  "e": "ORDER_TRADE_UPDATE",
  "T": 1689244751953,
}
```

```json=
{
  "excute": "ORDER_TRADE_UPDATE",
  "time": 1689244751953,
}
```

這兩種格式都希望轉成
```java=
@Data
public class DTO1 {
    private String excute;
    private Long time;
}
```

## 快速的解決辦法有兩個

### 方法一 利用@JsonAlias

```java=
@Data
public class DTO1 {
    @JsonAlias({"excute", "e"})
    private String excute;
    @JsonAlias({"time", "T"})
    private Long time;
}
```

### 方法二 利用 seter
```java=
@Data
public class DTO1 {
    private String excute;
    private Long time;
    
   public void setT(Long t) {
        this.time = t;
    } 
    
    public void setE(String e) {
        this.excute = e;
    }
        
}
```

### 利用方法二 可以解耦合
如果不想讓主要的DTO 寫太多不同情境的 CODE 可以改成如下
```java=
@Data
public class DTO1 {
    private String excute;
    private Long time;   
}

public class DTO2 extends DTO1{
   
   public void setT(Long t) {
        this.time = t;
    } 
    
    public void setE(String e) {
        this.excute = e;
    }

}
```

> 這樣DTO1 就可保持clean 直後要拔出來也必較方便