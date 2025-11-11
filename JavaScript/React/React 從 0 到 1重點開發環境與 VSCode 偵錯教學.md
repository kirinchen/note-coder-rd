### React 從 0 到 1：重點開發環境與 VSCode 偵錯教學

這篇教學的目標不是教完 React 的所有語法，而是幫您建立一個**最高效率的開發環境**。

#### 第 1 部分：環境安裝 (使用 Vite)

忘掉 `create-react-app` 吧！Vite 是現代 React 開發的首選，它提供了**秒級的啟動速度**和**即時熱更新 (HMR)**，體驗非常好。

**1. 安裝 Node.js**

React 開發依賴於 Node.js。請至 [Node.js 官網](https://nodejs.org/) 下載並安裝 **LTS (長期支援)** 版本。

安裝完成後，打開您的終端機 (Terminal 或命令提示字元) 並輸入以下指令，確認安裝成功：



```Bash
node -v
npm -v

```

您應該會看到各自的版本號。

**2. 建立您的第一個 React 專案**

這是在 React 18+ 時代，建立新專案的標準指令。



```Bash
npm create vite@latest

```

執行後，它會問您幾個問題：

1.  `? Project name:` (專案名稱)
    
    -   ➡️ 輸入 `my-react-app` (或您喜歡的任何名稱)
        
2.  `? Select a framework:` (選擇框架)
    
    -   ➡️ 使用上下鍵選擇 **React**
        
3.  `? Select a variant:` (選擇變體)
    
    -   ➡️ 選擇 **JavaScript** (如果您熟悉 TypeScript，也可以選 `TypeScript`)
        

完成後，Vite 會提示您接下來的步驟：



```Bash
Done. Now run:

  cd my-react-app
  npm install
  npm run dev

```

**3. 啟動您的專案**

讓我們跟著它的指示做：



```Bash
# 1. 進入您剛剛建立的資料夾
cd my-react-app

# 2. 安裝所有依賴的套件
npm install

# 3. 啟動開發伺服器
npm run dev

```

當您執行 `npm run dev` 後，您會看到類似這樣的訊息：

```
  VITE v5.x.x  ready in 310ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help

```

打開您的瀏覽器，訪問 `http://localhost:5173/`，您就會看到您的第一個 React App 正在運行！

----------

#### 第 2 部分：VSCode 整合與必備擴充套件

現在，使用 VSCode 打開 `my-react-app` 這個資料夾。



```Bash
# 在 my-react-app 資料夾中
code .

```

為了讓開發更順手，請在 VSCode 的「擴充套件」分頁 (Extensions) 安裝以下幾個工具：

1.  **ES7+ React/Redux/React-Native snippets** (作者: dsznajder)
    
    -   **必裝**。讓您能用 `rfce` (快速建立函式元件)、`useState` (快速插入 hook) 等快捷鍵。
        
2.  **Prettier - Code formatter** (作者: Prettier)
    
    -   程式碼自動排版工具，保持程式碼風格一致。
        
3.  **ESLint** (作者: Microsoft)
    
    -   即時檢查您的程式碼是否有語法錯誤或潛在問題。
        

----------

#### 第 3 部分：VSCode 偵錯設定 (核心重點)

> 下載Page : https://code.visualstudio.com/docs/?dv=win64user

這就是能讓您擺脫 `console.log` 大法的關鍵一步。我們要在 VSCode 中設定「中斷點 (Breakpoint)」，讓程式在特定行暫停。

**1. 為什麼需要設定？**

Vite 啟動的 React 專案是運行在瀏覽器中的。我們需要告訴 VSCode 如何將它自己連接到瀏覽器（如 Chrome 或 Edge）的偵錯程序上。

**2. 建立 `launch.json` 偵錯設定檔**

1.  點擊 VSCode 左側的「**執行與偵錯**」圖示 (Run and Debug，一個帶有播放鍵的蟲子)。
    
2.  您會看到 "Run and Debug" 按鈕，下面可能有一個連結「**建立 launch.json 檔案 (create a launch.json file)**」。
    
3.  點擊它。在跳出的選擇框中，選擇 **Web App (Chrome)**。
    
    -   (如果您主要使用 Edge，選擇 **Web App (Edge)** 也可以，兩者設定通用)。
        
4.  VSCode 會自動在您的專案中建立一個 `.vscode/launch.json` 檔案。
    

**3. 修改 `launch.json`**

自動產生的設定檔需要修改才能對應到 Vite 的 port。請將 `launch.json` 的內容**完全替換**為以下設定：



```JSON
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Chrome against localhost",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:5173", // 確保這個 port 與 'npm run dev' 的 port 一致
      "webRoot": "${workspaceFolder}"
    }
    // 如果您使用 Edge，可以新增這個 (或只用這個)
    // {
    //   "name": "Launch Edge against localhost",
    //   "type": "msedge",
    //   "request": "launch",
    //   "url": "http://localhost:5173",
    //   "webRoot": "${workspaceFolder}"
    // }
  ]
}

```

-   `"type": "chrome"`: 告訴 VSCode 我們要使用 Chrome 偵錯器。
    
-   `"url": "http://localhost:5173"`: 這是**最關鍵的一行**。它告訴 VSCode 要啟動並附加到 Vite 運行的 5173 埠。
    
-   `"webRoot": "${workspaceFolder}"`: 幫助 VSCode 將瀏覽器中的程式碼對應回您專案中的原始檔案 (Source Map)。
    

----------

#### 第 4 部分：開始您的偵錯流程

現在，讓我們來實際體驗這個流程：

1.  **第一步：啟動開發伺服器**
    
    -   在您的 VSCode **終端機** (Terminal) 中，確保 `npm run dev` 正在運行。
        
2.  **第二步：設定中斷點 (Breakpoint)**
    
    -   打開 `src/App.jsx` 檔案。
        
    -   Vite 預設的範本中有一個 `useState` hook：`const [count, setCount] = useState(0)`。
        
    -   在 App.jsx 中找到 onClick 函數的那一行，例如：
        
        onClick={() => setCount((count) => count + 1)}
        
    -   讓我們把它改成一個完整的函式，以便設定中斷點：
        
    
    
    
    ```JavaScript
    // ... 其他 import
    import './App.css'
    
    function App() {
      const [count, setCount] = useState(0)
    
      // ↓↓↓ 將 onClick 的邏輯抽出來變成一個函式 ↓↓↓
      function handleCountClick() {
        setCount((count) => count + 1) // <--- 在這一行的行號左邊點一下
      }
    
      return (
        <>
          {/* ... 其他 JSX ... */}
          <div className="card">
            <button onClick={handleCountClick}> {/* <--- 改用這個新函式 */}
              count is {count}
            </button>
            {/* ... 其他 JSX ... */}
          </div>
        </>
      )
    }
    
    export default App
    
    ```
    
    -   在 `setCount((count) => count + 1)` 這一行的**行號左側**點擊一下，您會看到一個**紅點**。這就是中斷點。
        
3.  **第三步：啟動偵錯器**
    
    -   回到左側的「**執行與偵錯**」分頁。
        
    -   在頂部的下拉選單中，選擇您剛剛設定的 **"Launch Chrome against localhost"**。
        
    -   按下旁邊的**綠色播放按鈕 (▶️)** (或按 `F5`)。
        
4.  **第四步：見證奇蹟**
    
    -   VSCode 將會自動**啟動一個全新的 Chrome 瀏覽器視窗**，並打開 `http://localhost:5173`。
        
    -   在這個**新的**瀏覽器視窗中，點擊 "count is 0" 那顆按鈕。
        
    -   **立刻切換回 VSCode**。
        
    
    您會發現：
    
    -   程式執行到 `setCount(...)` 這一行時**暫停**了！
        
    -   在 VSCode 左側的「變數 (VARIABLES)」視窗中，您可以看到目前所有的變數值 (例如 `count` 現在是 0)。
        
    -   您可以將滑鼠移到程式碼中的 `count` 變數上，也會彈出小視窗顯示它的值。
        
    -   您可以使用頂部的偵錯工具列（繼續、步過、步入...）來逐行執行程式碼。
        

### 總結

恭喜！您已經完成了 React 開發最重要的第一步：

1.  使用 **Vite** 建立了現代化的 React 專案。
    
2.  安裝了必備的 **VSCode 擴充套件**。
    
3.  設定了 `launch.json`，讓 VSCode 可以**直接偵錯**在 Chrome 中運行的 React 程式碼。
    

您現在擁有了一個專業的開發環境，可以告別 `console.log`，開始高效地學習和開發 React 了！