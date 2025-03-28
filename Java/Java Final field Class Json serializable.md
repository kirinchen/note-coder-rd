---
title: Java Final field Class Json serializable
tags: [java, jackson]

---

code :

```java=
@Jacksonized
@Builder
public class BeginEndIdx {

    public final int begin;
    public final int end;


    public static void main(String[] args) {
        BeginEndIdx beginEndIdx = new BeginEndIdx(1, 2);
        String json = JsonUtils.toJson(beginEndIdx);
        System.out.println(json);
        BeginEndIdx beginEndIdx1 = JsonUtils.loadByJson(json, BeginEndIdx.class);
        System.out.println(beginEndIdx1);
    }
}
```

output 
```shell=
{"begin":1,"end":2}
net.bzk.infrastructure.obj.BeginEndIdx@65d6b83b
```