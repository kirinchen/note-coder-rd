---
title: Java lombok 好用annotation小筆記
tags: [java]

---

# Java lombok 好用annotation小筆記


## 建構子設定

```java
@AllArgsConstructor(access = AccessLevel.PUBLIC)
```
* AccessLevel --> 用來設定建構子的可視的等級  public private ...

* EX

```java
@Data
@AllArgsConstructor(access = AccessLevel.PUBLIC)
public class RegisteredFlow {

	private int flow;
	private List<RunInfo> runInfos;

	@Data
	@AllArgsConstructor(access = AccessLevel.PUBLIC)
	public static class RunInfo {
		private String runUid;
		private State state;
		private String endTag;
		private String endLinkUid;
	}
	
	public static void main(String[] args){
	    List<RunInfo> l = new List<RunInfo>();
	    RegisteredFlow rf = new RegisteredFlow(123,l);
	}
}

```

> 如果還要包含 空建構子, 可以再加入 **@NoArgsConstructor**

```java=
@Data
@AllArgsConstructor(access = AccessLevel.PUBLIC)
@NoArgsConstructor
public class RegisteredFlow {
```

## builder pattern

關鍵字 : ``` 	@Builder  ```

範例:
```java
	@Data
	@Builder
	public class Bundle {
		private String runFlowUid;
		private String flowUid;
		private FlowRuner flowRuner;

        public static void main(String[] args){
            Bundle ans = BoxRuner.Bundle.builder().flowUid(model.getUid()).runFlowUid(uid).flowRuner(this).build();
        }

	}
```

### 有繼承並且 Json Seriable
```java=

import lombok.experimental.SuperBuilder;
import lombok.extern.jackson.Jacksonized;
import net.bzk.infrastructure.JsonUtils;

@SuperBuilder
@Jacksonized
public class ExcReq {
    public final String beanName;
    @SuperBuilder()
    @Jacksonized
    public static class TryExClz extends ExcReq{
        public final String ex;
    }

    public static void main(String[] args) {
        TryExClz tc = TryExClz.builder()
                .beanName("12")
                .ex("xx")
                .build();
        String json= JsonUtils.toJson(tc);
        System.out.println(json);
        TryExClz tc2 = JsonUtils.loadByJson(json,TryExClz.class);
        System.out.println(tc2);
    }
}
```

### 如果想要其中某個 field 不要被設定
```java=
@Builder
public class InfluxPoint {
    @Builder.Default
    public Map<String, String> tags = new HashMap<>();
    public String field;
    public double val;
}
```

## @Data 設定某些欄位 geter seter 不要出現
* 關鍵字 : 
```java
@Getter(value = AccessLevel.NONE)
@Setter(value = AccessLevel.NONE)
```
 * ex:
```java
	@Data
	public class Bundle {
		private String runFlowUid;
		private String flowUid;
		@Getter(value = AccessLevel.NONE)
        @Setter(value = AccessLevel.NONE)
		private FlowRuner flowRuner;
	}
```

## 屬性自動產生geter 或 seter

關鍵字
```java
@Getter
@Setter
```
 * ex:
```java
	
	public class Bundle {
		private String runFlowUid;
		@Getter
        @Setter
		private String flowUid;
	}
```

> 這樣不需要@Data 就可以對某個屬信產生 geter 或 seter

## 自動產生 Slf4j logger
關鍵字
```java
@Slf4j
```
 * ex:
```java
	
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class NodejsActionCall  {
	private void runCmd(Consumer<String> c, String... cmd) {
		log.info("runCmd:" + cmd);
		...
	}

}

```

## 持續更新...


###### tags: `java`





