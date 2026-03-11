
### 第一階段：打造 Tauri v2 專案基礎


### Step 1: 環境最終確認 (Windows 專屬避坑指南)

在 Windows 上開發 Tauri，你的系統必須同時具備 **Node.js (管前端)**、**Rust (管後端)**，以及最重要的 **C++ Build Tools (管底層編譯)**。請嚴格按照以下三個子步驟檢查你的電腦：

#### 1.1 安裝正牌的微軟編譯器 (Visual Studio C++ Build Tools)

Tauri 底層需要把 Rust 程式碼組裝成 Windows 原生的 `.exe`，這個「組裝」的動作必須依賴微軟的 C++ 連結器（`link.exe`）與 Windows SDK。

1. 下載並執行 [Visual Studio C++ Build Tools](https://www.google.com/search?q=https://visualstudio.microsoft.com/visual-cpp-build-tools/) 安裝程式。
    
2. 在安裝介面中，勾選左側的 **「使用 C++ 的桌面開發」 (Desktop development with C++)**。
    
3. **⚠️ 極致瘦身設定**：微軟預設會幫你勾選一堆用不到的肥大工具。請看右側的「安裝詳細資料」，**只保留以下兩項打勾，其餘通通取消**：
    
    - ✅ **適用於 x64/x86 的最新 Microsoft Visual C++ 建置工具** (這包含最重要的 MSVC 編譯器)
        
    - ✅ **Windows 11 SDK** (或 Windows 10 SDK，提供系統底層 API)
        
4. 確認右下角安裝大小約在 2GB ~ 3GB 左右，點擊安裝並等待完成。
    

#### 1.2 確保 Rust 工具鏈已正確就緒

有了微軟的底層工具後，我們才能安裝 Rust，讓 Rust 安裝程式能順利抓到微軟的編譯器。

1. 如果你還沒安裝 Rust，請到 [rustup.rs](https://rustup.rs/) 下載 `rustup-init.exe`，一路按 Enter 預設安裝到底。
    
2. 打開終端機（請用 Windows 內建的 CMD 或 PowerShell），輸入指令檢查：
    
    
    
    ```Bash
    rustc --version
    cargo --version
    ```
    
    如果有印出版本號（例如 `rustc 1.x.x`），代表 Rust 安裝成功。
    

#### 1.3 徹底封殺 Git Bash 的「撞名陷阱」(終端機設定)

這是最多人踩坑的地方！Git Bash 裡面自帶了一個 Linux 移植過來的同名工具 `link.exe`。如果你在 Git Bash 裡面執行 Tauri 編譯，系統會抓錯工具，導致噴出 `Try 'link --help' for more information` 的詭異錯誤。

**在你的編輯器 (VS Code 或 Cursor) 中，請務必切換終端機：**

1. 按下快捷鍵 `Ctrl` + `Shift` + `P` 打開命令面板。
    
2. 搜尋並選擇 **`Terminal: Select Default Profile` (終端機: 選取預設設定檔)**。
    
3. 在跳出的選單中，明確選擇 **`PowerShell`** 或 **`Command Prompt`**。
    
4. **把目前開著的終端機視窗全部按垃圾桶關掉**，然後按下 `` Ctrl + ` `` 開啟一個全新的 PowerShell 終端機。
    
5. _(可選終極保險)_：如果你想避開所有環境變數的問題，可以直接點擊 Windows 開始按鈕，搜尋並打開 **「x64 Native Tools Command Prompt for VS 2022」**，在這個微軟專屬的純淨黑框框裡進行後續開發。
    


#### Step 2: 初始化專案

在終端機執行 Tauri 官方的快速建置指令。我們將專案命名為 `inlay-hello`：



```Bash
npm create tauri-app@latest
```

**請依照提示進行以下選擇：**

1. Project name: `inlay-hello`
    
2. Choose which language to use for your frontend: `TypeScript / JavaScript`
    
3. Choose your package manager: `npm`
    
4. Choose your UI template: `React`
    
5. Choose your UI flavor: `TypeScript`
    

建立完成後，進入資料夾並安裝前端套件：



```Bash
cd inlay-hello
npm install
```

---

### 第二階段：實作 Rust 後端 API (Command)

Tauri 的精神是「前端負責 UI，後端負責重度運算與系統權限」。我們現在要在 Rust 寫一支 API，讓前端的 React 可以呼叫它。

請打開 `src-tauri/src/lib.rs`，把內容替換成以下乾淨的程式碼：



```Rust
// src-tauri/src/lib.rs

// 1. 加上巨集標籤，告訴 Tauri 這個函數要暴露給前端
#[tauri::command]
fn greet(name: &str) -> String {
    // 使用 println! 巨集，將訊息印在你的開發終端機 (Backend Console)
    println!("【Rust 後端】收到前端傳來的名字: {}", name);
    
    // 組合字串。結尾不加分號，代表將這行作為結果 return 給前端
    format!("Hello, {}! 歡迎來到 Tauri v2 x Rust 的極速世界！", name)
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        // Tauri v2 預設的系統開啟工具外掛
        .plugin(tauri_plugin_opener::init()) 
        // 2. 在這裡註冊你的 greet 函數，前端的 invoke 才找得到對應的通道
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

### 第三階段：實作 React 前端呼叫 (IPC)

後端寫好了，現在我們來到熟悉的前端領域。我們將使用 Tauri 提供的 `invoke` 方法，透過 IPC (進程間通訊) 安全地呼叫 Rust 函數。

請打開 `src/App.tsx`，把內容清空並替換為以下程式碼：



```TypeScript
// src/App.tsx
import { useState } from "react";
// 引入 Tauri 的核心 API，用來呼叫 Rust 函數
import { invoke } from "@tauri-apps/api/core";
import "./App.css";

function App() {
  const [name, setName] = useState("");
  const [greetMsg, setGreetMsg] = useState("");

  async function handleGreet() {
    // 這裡就像在打後端 API 一樣，"greet" 必須對應 Rust 裡面的函數名稱
    // 傳遞的物件 key 也必須對應 Rust 函數的參數名稱 (name)
    try {
      const response = await invoke<string>("greet", { name });
      setGreetMsg(response);
    } catch (error) {
      console.error("呼叫 Rust 發生錯誤:", error);
    }
  }

  return (
    <div style={{ padding: "40px", fontFamily: "sans-serif", textAlign: "center" }}>
      <h1>Inlay Workspace (Tauri v2)</h1>
      
      <div style={{ marginTop: "30px", display: "flex", justifyContent: "center", gap: "10px" }}>
        <input
          type="text"
          placeholder="請輸入你的名字..."
          value={name}
          onChange={(e) => setName(e.currentTarget.value)}
          style={{ padding: "10px", fontSize: "16px", borderRadius: "6px", border: "1px solid #ccc" }}
        />
        <button 
          onClick={handleGreet} 
          style={{ padding: "10px 20px", fontSize: "16px", cursor: "pointer", borderRadius: "6px" }}
        >
          發送給 Rust
        </button>
      </div>

      {/* 顯示從 Rust 回傳的結果 */}
      {greetMsg && (
        <p style={{ marginTop: "30px", fontSize: "18px", color: "#007acc", fontWeight: "bold" }}>
          {greetMsg}
        </p>
      )}
    </div>
  );
}

export default App;
```

---

### 第四階段：啟動並見證奇蹟

回到你的終端機（再次確認是 PowerShell 或 CMD），執行啟動指令：



```Bash
npm run tauri dev
```

這一次，因為環境已經完全乾淨，你會看到 Vite 瞬間啟動，接著 Rust 快速編譯。幾秒鐘後，一個原生的作業系統視窗就會彈出來。

在輸入框打上文字並按下按鈕，你不僅會在 UI 畫面上看到藍色的歡迎字串，**低頭看一下你的終端機**，也會看到 Rust 成功印出 `【Rust 後端】收到前端傳來的名字: ...`！
