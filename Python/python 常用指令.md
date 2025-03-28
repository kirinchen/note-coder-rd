---
title: python 常用指令
tags: [python]

---

# python 常用指令


## pipenv 

### 進入env環境
```bash=
pipenv shell
```

### 產生 requirements.txt
```bash=
pipenv lock --requirements > requirements.txt
```

### Pipfile install local repository

```bash=
pipenv install -e ${local repository path} --skip-lock
```

Example
```bash=
pipenv install -e /Users/g01880/mop_api_core --skip-lock
```



## pip 相關的

* 列出目前所有moudles
```bash=
pip freeze 
```

## pyenv

* 列出可安裝的 Python 版本的List

```bash=
pyenv install -l
```


###### tags: `python`