---
title: 調效能神器 JDK Mission Control (JMC)  連接設定 + 監控Docker下微服務的設定
tags: [java]

---

# 調效能神器 JDK Mission Control (JMC)  連接設定 + 監控Docker下微服務的設定 

## 先設定 JMX 在執行Java Application 增加以下參數

```java=
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=7091 
-Dcom.sun.management.jmxremote.rmi.port=7091 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false 
-Djava.rmi.server.hostname=localhost
```

ex
```shell=
java   \
-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.port=7091 \
-Dcom.sun.management.jmxremote.rmi.port=7091 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false \
-Djava.rmi.server.hostname=localhost \
  -jar Notepad.jar
```

### Docker上執行請看以下設定

#### 修改DockerFile

```dockerfile=


...

EXPOSE 8080 7091

ENTRYPOINT ["java","-Dspring.profiles.active=prod","-Dcom.sun.management.jmxremote","-Dcom.sun.management.jmxremote.port=7091","-Dcom.sun.management.jmxremote.rmi.port=7091","-Dcom.sun.management.jmxremote.authenticate=false","-Dcom.sun.management.jmxremote.ssl=false","-Djava.rmi.server.hostname=localhost","-jar","/app/spring-boot-application.jar"]
```

#### 啟動方式

以 **docker-compose** 設定為例
```yml
  xxxx:
    build:
        context: ./xxx/
        dockerfile: xxx/Dockerfile
    ports: 
      - "8080:8080"
      - "7091:7091"
```
增加新的7091

## 利用 SSH + XShell Tunnel 當成VPN
因為如果開成外網可以連線是相當危險的事,設定authenticate也麻煩且還是有危險性,我們可以利用SSH當VPN直接
連接過去

以下是用 [XSell](https://www.netsarang.com/en/xshell/) SSH連接程式裡面的 Tunnel功能

進入點 : Sessions > [目標的連結]> Properties > Tunnel 

![](https://i.imgur.com/qow18nq.png)




## JMC 連接

* 執行 `[java home]/bin/jmc.exe`
* 連接

![](https://i.imgur.com/ERTNIPo.png)
* Enjoy
![](https://i.imgur.com/f25ocRH.png)



## other

`[java home]/bin/jmc.exe` 
還有很多可以使用build-in JMX 工具,或是其他第三方的JMX工具可以使用
本筆記只是用JMC作範例

###### tags: `java`