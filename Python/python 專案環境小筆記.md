---
title: python 專案環境小筆記
tags: [python]

---

# python 專案環境小筆記

1. 管理python版本 - pyenv : https://blog.codylab.com/python-pyenv-management/
2. 管理套件版本 - pipenv : https://medium.com/@chihsuan/pipenv-%E6%9B%B4%E7%B0%A1%E5%96%AE-%E6%9B%B4%E5%BF%AB%E9%80%9F%E7%9A%84-python-%E5%A5%97%E4%BB%B6%E7%AE%A1%E7%90%86%E5%B7%A5%E5%85%B7-135a47e504f4




## 一些小坑 除雷經過

### pyenv 安裝路徑
```bash=
C:\Users\DDT\.pyenv\pyenv-win\versions\3.7.9
```

### poetry path
```bash=
$ which poetry
/c/Users/DDT/AppData/Local/Programs/Python/Python39/Scripts/poetry

```

### 如果要讓pipenv 吃pyenv 的版本
直接指定路徑 pyenv 安裝python 的 install path


> pipenv path ex: C:\Users\DDT\AppData\Local\Programs\Python\Python39\Scripts

pipenv install --python "安裝路徑"

>Example :
```bash=
pipenv install --python "C:\Users\g02042\.pyenv\pyenv-win\versions\3.6.8\python.exe"
```

### pycharm 如果pipenv python version 不一致

pycharm去 Settings > Poject > Python Interpreter 指定版本為 pipenv , 如果環境被改的很混亂,建議可以去 vitrvalenv 把環境都刪除

### 在PyCharm 中執行遇到 .env 變數讀不到的問題

![](https://i.imgur.com/OqV8Fy0.png)

![](https://i.imgur.com/O5Zw9mE.png)

直接把 .env 內容貼上即可

## 如何吃本地端的 moudle

### 方法一 直接改 pipfile

```javascript=
xxx-package = {editable = true, path = "C:/Users/g02042/Desktop/Projects/mop_enum"}
```

### 方法二 下 pipenv 指令

###### tags:  `python`