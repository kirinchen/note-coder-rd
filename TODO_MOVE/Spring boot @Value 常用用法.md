---
title: Spring boot @Value 常用用法
tags: [java, springboot]

---

# Spring boot @Value 常用用法

## 基本使用 ${變數}
直接抓.properties 變數

## 沒有抓為null ${變數:#{null}}
直接抓.properties 變數 預設為 null , 這樣沒有也不會爆錯
```java=
    @Value("${fin_exchange_manage.exchange_account:#{null}}")
    private String defaultExchangeAccount = null;
```

## 沒有抓為字串 ${ccc.bbb:}
```java=
	@Value("${ccc.bbb:}")
	private String defaultExchangeAccount = null;
```

## 抓取 Resource 檔案
```java=
    @Value("classpath:config/task_back_trade.js")
    private Resource configJsRes;
```


###### tags: `java` `springboot`