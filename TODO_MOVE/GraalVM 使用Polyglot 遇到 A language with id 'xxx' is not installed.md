---
title: GraalVM 使用Polyglot 遇到 A language with id 'xxx' is not installed
tags: [devops, java, GraalVM]

---

# GraalVM 使用Polyglot 遇到 A language with id 'xxx' is not installed

錯誤訊息 example:
```
Exception in thread "main" java.lang.IllegalArgumentException: A language with id 'xxx' is not installed. Installed languages are: [js].
	at com.oracle.truffle.polyglot.PolyglotEngineImpl.requirePublicLanguage(PolyglotEngineImpl.java:695)
	at com.oracle.truffle.polyglot.PolyglotContextImpl.requirePublicLanguage(PolyglotContextImpl.java:821)
	at com.oracle.truffle.polyglot.PolyglotContextImpl.eval(PolyglotContextImpl.java:792)
	at org.graalvm.polyglot.Context.eval(Context.java:341)
	at org.graalvm.polyglot.Context.eval(Context.java:367)
	at org.vena.finch.FinchApplication.main(FinchApplication.java:12)
```

此時要去安裝 truffle js plugin

## 安裝 truffle language 套件

像是我需要的是 'js' 套件

```shell
$<GraalVM路徑>/bin/gu install js
```

> 此方始只支援 GraalVM 22.2以上

## 重啟應用程式即可

enjoy!!

[參考官方文件 ](https://www.graalvm.org/22.2/reference-manual/js/)

###### tags: `devops` `java` `GraalVM` 