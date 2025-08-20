# 告別龜速！用 Vite 光速啟動你的 React 新專案 (2025 最新教學)

**發佈日期：** 2025年8月6日

你是否還在使用 `create-react-app` (CRA) 建立新的 React 專案，然後花費數分鐘甚至更久的時間等待所有依賴安裝完畢？是時候告別龜速的開發體驗，擁抱新一代的前端建構工具——**Vite** 了！

Vite (法文，音同 "veet"，意為「快速」) 是一個革命性的前端工具，它利用瀏覽器原生的 ES 模組 (ESM) 功能，實現了**秒級的專案啟動速度**和**即時的熱模組更新 (HMR)**。無論你的專案規模多大，Vite 都能提供閃電般的回饋，讓你的開發流程如絲般滑順。

這篇教學將帶你一步步從零開始，使用 Vite 建立一個全新的 React + TypeScript 專案。讓我們開始吧！

## ## 事前準備 (Prerequisites)

在開始之前，請確保你的電腦上已經安裝了 **Node.js**。Vite 需要 Node.js 版本 `18+` 或 `20+`。安裝 Node.js 的同時，也會自動安裝好套件管理工具 **npm**。

你可以打開終端機 (Terminal) 並輸入以下指令來檢查你的版本：

```
node -v
npm -v
```

如果版本符合要求，你就可以開始了！

## ## 第一步：建立 Vite 專案

這一步非常簡單。打開你的終端機，移動到你想要建立專案的資料夾，然後執行以下指令：

```
npm create vite@latest
```

- **小提示：** 如果你習慣使用 `yarn` 或 `pnpm`，也可以執行 `yarn create vite` 或 `pnpm create vite`。
    

執行指令後，Vite 會以互動問答的方式引導你完成設定：

1. **✔ Project name: …** `› my-react-app`
    
    - 輸入你的專案名稱，例如 `my-react-app`，然後按下 Enter。
        
2. **✔ Select a framework: …**
    
    - 使用上下方向鍵選擇 **React**，然後按下 Enter。
        
3. **✔ Select a variant: …**
    
    - 強烈建議選擇 **TypeScript**（或 `TypeScript + SWC`，速度更快）。在現代前端開發中，TypeScript 能提供絕佳的型別安全和開發體驗。選擇後按下 Enter。
        

完成後，你會看到一個成功的訊息，告訴你專案已經建立好了！

```
Done. Now run:

  cd my-react-app
  npm install
  npm run dev
```

## ## 第二步：進入專案並安裝依賴

根據上一步的提示，我們需要先進入剛剛建立的專案資料夾，然後安裝所有必要的依賴套件。

```
# 1. 進入專案資料夾
cd my-react-app

# 2. 安裝依賴
npm install
```

這個安裝過程通常也比傳統的 `create-react-app` 快上許多，因為 Vite 的依賴本身就非常輕量。

## ## 第三步：啟動開發伺服器

安裝完畢後，就可以啟動 Vite 的開發伺服器了。這一步的速度會讓你感到驚艷。

```
npm run dev
```

執行後，你幾乎是立刻就會在終端機看到類似以下的訊息：

```
  VITE v5.3.1  ready in 315 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

看到了嗎？只花了 **315 毫秒**！這就是 Vite 的威力。

現在，用你的瀏覽器打開 `http://localhost:5173/`，你應該能看到一個預設的 React 歡迎頁面，上面有一個可以點擊計數的按鈕。

## ## 專案結構速覽

Vite 專案的結構非常直觀和現代化。

```
my-react-app/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.css
│   ├── App.tsx       # 主要的 App 元件
│   ├── index.css
│   └── main.tsx      # 專案的進入點
├── .eslintrc.cjs
├── .gitignore
├── index.html        # 重要的入口 HTML 檔案
├── package.json
├── tsconfig.json
└── vite.config.ts    # Vite 的設定檔
```

**幾個關鍵點：**

- **`index.html` 在根目錄：** 與 CRA 不同，`index.html` 是專案的真正入口，而不是放在 `public` 資料夾。Vite 會以它為基礎來建構你的應用。
    
- **`src/main.tsx`：** 這是 JavaScript 的進入點，它會找到 `index.html` 中的 `<div id="root"></div>`，並將你的 React App 渲染進去。
    
- **`vite.config.ts`：** 所有 Vite 相關的設定，例如代理 (proxy)、路徑別名 (alias) 等，都會在這裡配置。
    

## ## 結論

恭喜你！你已經成功使用 Vite 建立並啟動了一個現代化的 React 專案。

從今天起，你可以徹底忘掉 `create-react-app`，全面擁抱 Vite 帶來的極致開發體驗。現在，試著修改 `src/App.tsx` 檔案中的內容，然後看看瀏覽器是不是在你儲存檔案的瞬間就更新了畫面。

享受 coding 吧！

## VScode lanch.json
```json
{

    // Use IntelliSense to learn about possible attributes.

    // Hover to view descriptions of existing attributes.

    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387

    "version": "0.2.0",

    "configurations": [

        {

            "type": "chrome",

            "request": "launch",

            "name": "Launch Chrome against localhost",

            "url": "http://localhost:5173",

            "webRoot": "${workspaceFolder}"

        }

    ]

}
```