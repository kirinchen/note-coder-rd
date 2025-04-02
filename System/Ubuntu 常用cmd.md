---
title: Ubuntu 常用cmd
tags: [devops, linux]

---

# Ubuntu 常用cmd

## Java

### 顯示安裝路徑

```bash=
readlink -f $(which java) 
```

## Task Process

### 用關鍵字查詢執行的 ID
```bash=
ps -ef|grep [關鍵字]
```

or 
```bash=
pgrep -f xyz.jar
```

### nohup 離線或登出系統後繼續執行

* [參考] https://blog.gtwang.org/linux/linux-nohup-command-tutorial/
 ex:
```bash=
nohup /path/my_program > my.out 2> my.err &
```

### kill -p by app name
```bash=
#!/bin/sh
if apid=$(pgrep -f xyz.jar)
then
    echo "Running, pid is $apid"
    kill -9 xyz.jar
else
    echo "Stopped"

fi
```

### 設定啟動程式和java path
```shell
sudo vi /etc/profile
```
Add following lines in end
```shell=
JAVA_HOME=/usr/lib/jvm/jdk1.7.0
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export JRE_HOME
export PATH
```

### 尋找指令實際位置
```shell=
$ readlink -f $(which javac)
/usr/lib/jvm/java-14-openjdk-amd64/bin/javac
$ readlink -f $(which java)
/usr/lib/jvm/java-14-openjdk-amd64/bin/java
```

###### tags: `devops` `linux`