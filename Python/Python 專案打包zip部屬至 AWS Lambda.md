---
title: Python 專案打包zip部屬至 AWS Lambda
tags: [devops, aws, python]

---

# Python 專案打包zip部屬至 AWS Lambda

雖然現在基本都是透過 Docker 搭配 Aws ECR ,但是有時候只是想快速丟到Lambda跑跑看

就可以看這篇教學


## 複製引用套件

1. 如過是pipenv的專案 可用以下方式轉成 **requirements.txt**
```shell=
pipenv lock -r >requirements.txt
```
2. 把引用的套件複製出來
```shell=
docker run -v "<專案路徑>":/var/task "lambci/lambda:build-python3.8" /bin/sh -c "pip install -r requirements.txt -t python/lib/python3.9/site-packages/; exit"
```

> 使用這個指令 需要先安裝docker
> 專案路徑需使用絕對路徑

3. 會將所以的套件複製到 python/lib/python3.9/site-packages/ 目錄下
![](https://i.imgur.com/GcGvotK.png)


## 打包zip

1. 將"套件"的資料夾移動到 ../dest目錄
```shell=
mkdir ../dest
mv ./python/lib/python3.9/site-packages/* ../dest 
rm -rf ./python/
```

2. 複製專案內的檔案到../dest 
```shell=
cp -r * ../dest
```
3. 用zip把 ../dest的專案打包巴 就可以上傳了

### enjoy!

###### tags: `devops` `aws` `python`