---
title: JPA Create/Update 前修改 Entity (@PrePersist/@PreUpdate)
tags: [java, jpa]

---

# JPA Create/Update 前修改 Entity (@PrePersist/@PreUpdate)

常常我們需要再寫入資料庫前,固定寫入某寫欄位 例如 創建日期 更新日期

## 原本操作方式


Model - Account.java
```java=
import lombok.Data;
import javax.persistence.*;

@Data
@Entity
public class Account implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String username;
    private String password;
    private Date createAt;
    private Date updateAt;    
}    
```


Service - AccountService.java
```java=
public class AccountService {

    ...
    public Account createAccount(String user,String pass){
        Account a = new Account();
        ...
        Date nowAt = CommUtils.nowUtc0();
        a.setCreateAt(nowAt);
        a.setUpdateAt(nowAt);
        return dao.save(om);
    }
    
} 
```

## 改成 @PrePersist/@PreUpdate 去寫

創建一個新的

ModelAopService.java
```java=

public class ModelAopService {

    ...
    @PrePersist
    protected void onCreate(Account i) {
        Date now = CommUtils.nowUtc0();
        i.setCreateAt(now);
        i.setUpdateAt(now);
        i.setEnvProfile(springProfiles);
    }

    @PreUpdate
    protected void onUpdate(Account i) {
        i.setUpdateAt(new Date());
    }
    
} 

```

將原本的 Service - AccountService.java
```java=
public class AccountService {

    ...
    public Account createAccount(String user,String pass){
        Account a = new Account();
        ...
        return dao.save(om);
    }
    
} 
```
這樣就可以把固定的寫入code 搬過去了

## 如果有多個Model都有這個屬性

此寫法也有支援多型

改法如下 先創建 CreateUpdateInfo.java interface

```java=
public interface CreateUpdateInfo {

	public Date getUpdateAt() ;

	public void setUpdateAt(Date updateAt);

	public Date getCreateAt() ;

	public void setCreateAt(Date createAt) ;
	
}
```

 Model - Account.java impl CreateUpdateInfo
```java=
@Data
@Entity
public class Account implements Serializable,CreateUpdateInfo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String username;
    private String password;
    private Date createAt;
    private Date updateAt;    
}    
```

將 ModelAopService.java 改為
```java=

public class ModelAopService {

    ...
    @PrePersist
    protected void onCreate(CreateUpdateInfo i) {
        Date now = CommUtils.nowUtc0();
        i.setCreateAt(now);
        i.setUpdateAt(now);
        i.setEnvProfile(springProfiles);
    }

    @PreUpdate
    protected void onUpdate(CreateUpdateInfo i) {
        i.setUpdateAt(new Date());
    }
    
} 
```

之後只要實作CreateUpdateInfo 就會寫入必要資訊

