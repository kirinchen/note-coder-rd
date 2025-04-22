---
title: Git 如何記住帳號密碼
tags: [git]

---

# Git 如何記住帳號密碼

1. 到 project root 目錄
2. 找到 .git/config 檔案
3. 編輯該檔案
4. 增加以下設定
```yml
[credential]
  helper = store
```


###### tags: `git`