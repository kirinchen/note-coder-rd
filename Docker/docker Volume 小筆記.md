---
title: docker Volume 小筆記
tags: [docker]

---

# docker Volume 小筆記

## 查看Volume list

```bash=
docker volume ls
```

## 查看 Docker 目前使用空間

```bash=
docker system df
```

### 查看Volume 空間

```bash=
docker system df -v
```

### 清除未使用Volume

```bash=
docker volume prune
```

#### 強制清除

```bash=
docker volume prune -f
```

## 備份/還原 Volume


> 記得請先 stop container

### 將Volume備份到 /tmp/[匯出filename].tar.bz2

```bash=
docker run --rm -v [你的Volum名稱]:/volume -v /tmp:/backup alpine tar -cjf /backup/[匯出filename].tar.bz2 -C /volume ./
```

### 將Volume還原

```bash=
docker run --rm -v [你的Volum名稱]:/volume -v /tmp:/backup alpine sh -c "rm -rf /volume/* /volume/..?* /volume/.[!.]* ; tar -C /volume/ -xjf /backup/[匯出filename].tar.bz2"
```

## 常用CMD

### 刪除by name

```bash=
docker volume rm {名稱/id}
```

###### tags: `docker`
