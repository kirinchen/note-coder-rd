---
title: React web 透過 github action 部屬到firebase hosting 架設網站
tags: [React]

---

# React web 透過 github action 部屬到firebase hosting 架設網站

## github project 建立
先去 https://github.com/ 建立一個專案,然後要把 user/repository 記起來
``` ex : kirinchen/firebase-react-demo ```
然後我們找個目錄把專案clone下來

## react project 建立

```shell=
npx create-react-app firebase-react-demo --template typescript
```

> 先決條件要先安裝 npm & nodejs
> 可以參考這裡 [為什麼用React？什麼是Create-React-App？](https://ithelp.ithome.com.tw/articles/10234237)

先到該專案啟動測試看看
```shell=
cd firebase-react-demo
npm start
```

console output
```shell=
  http://localhost:3000

Note that the development build is not optimized.
To create a production build, use npm run build.

webpack compiled successfully
No issues found.
```
打開瀏覽器 type http://localhost:3000
![](https://i.imgur.com/tG4KOc7.png)
*這樣就成功了!!*



## firebase hosting 設定

先到 https://console.firebase.google.com/ 新增一組 firebase project

再回到專案cmd下
如果沒有安裝過 firebase cli 請安裝
```shell=
npm install -g firebase-tools
```
登入 Google
```shell=
firebase login
```

啟動初始化專案：
```shell=
firebase init
```

用方向鍵選擇到 然後用 空白鍵確定
![](https://i.imgur.com/rEYeR1f.png)
```Hosting: Configure files for Firebase Hosting and (optionally) set up GitHub Action deploys```

然後選擇 (用方向鍵選擇)
```shell=
Use an existing project
```
選擇剛剛建立的專案
之後選擇如下
```bash=
? What do you want to use as your public directory? build
...
? Set up automatic builds and deploys with GitHub? Yes
...

i  Authorizing with GitHub to upload your service account to a GitHub repository's secrets store.

Visit this URL on this device to log in:
https://github.com/login/oauth/authorize?client_id=xxx

Waiting for authentication...

+  Success! Logged into GitHub as kirinchen

? For which GitHub repository would you like to set up a GitHub workflow? (format: user/repository) kirinchen/firebase-react-demo

? Set up the workflow to run a build script before every deploy? Yes
? What script should be run before every deploy? npm run build

+  Created workflow file ...your site's live channel when a PR is merged? Yes
? What is the name of the GitHub branch associated with your site's live channel? main
...
+  Firebase initialization complete!
```
> 其中的 *GitHub repository would you like to set up a GitHub workflow* 是剛才我們設定的githut project

建立好後去project目錄下 *.github/workflows/* 編輯
* firebase-hosting-merge.yml
* firebase-hosting-pull-request.yml

在原本steps **- run: npm run build** 上面增加 **- run: npm install**
```yaml=
...
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm run build
      - uses: FirebaseExtended/action-hosting-deploy@v0
...      
```

## github action CI 部屬

這時候只要在該目錄下

```shell=
git add .
git commit -m "message"
git push -u origin main
```
> git 相關可以參考 https://ithelp.ithome.com.tw/articles/10271811

這時候就部署好了 我們去github action 的頁面就可以看到部屬的log
![](https://i.imgur.com/469fsAm.png)

這時候去 [firebase](https://console.firebase.google.com/project/) 找到剛建立的project下,點選 `hosting` 服務
![](https://i.imgur.com/wUcGPJO.png)
點選任一個連結 就可以看到我們剛透過 react create 建立的網站了
![](https://i.imgur.com/wz1hf09.png)


**Enjoy!**

### 範例
* [專案位置](https://github.com/kirinchen/firebase-react-demo)
* [範例網站](https://f-h-r-demo.web.app/)

###### tags: `React`