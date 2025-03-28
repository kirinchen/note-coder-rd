---
title: Docker 常用指令
tags: [docker]

---

# Docker 常用指令

## Stop所有的 container

```bash=
docker stop $(docker ps -a -q)
```
>  windows git Bash  /  Linux Only


## 刪除所有的 container

```bash=
docker rm $(docker ps -a -q)
```
>  windows git Bash  /  Linux Only
## 刪除所有的 Images
```bash=
docker rmi $(docker images -a -q)
```
>  windows git Bash  /  Linux Only

## 刪除container by name
```bash=
docker rm $( docker stop $(docker ps -a -q  --filter ancestor=<container name>))
```

## 刪除Image by name

```bash=
docker rmi --force $(docker images -q 'domi/sensor-raw-query' | uniq)
```

## 建置Image

```bash=
docker build -t {Tag名稱} .
```

## 修改已經建置好的 image tag and name

```shell
docker image tag d583c3ac45fd myname/server:latest
```


## 上傳image到 docker hub

```bash=
docker push yourhubusername/imagename
```

## 下載 image 從 docker hub
```bash=
docker pull yourhubusername/imagename
```

## 執行docker
```bash=
docker run --name {自行命名} -d -p 8080:80 {images name}
```
## 進入container 
```bash=
docker exec -it  container_name  /bin/sh
```

## 清除沒在使用的 Volume
```bash=
docker volume prune -f
```

## 清除沒有用的 image
```bash=
docker image prune -f
```

## 清除系統cache log bla bla
```bash=
docker system prune -af
```

## 清理 24h 沒有使用的 container
```shell=
docker container prune --filter "until=24h"
```

## Copy Docker裡的檔案
```bash=
docker cp $(docker ps -a -q  --filter name=bzk-tig_influxdb_1):/ifxdbback ./ifxdbback
```


###### tags: `docker`