# VS Code 偵錯設定指南 (`.vscode/launch.json`)

  

這份文件解釋了本專案中 `.vscode/launch.json` 檔案的作用。這個檔案用於設定 VS Code / Cursor 的偵錯 (Debug) 環境，讓我們可以直接在編輯器中執行和偵錯 Rust 程式碼。

  

## 為什麼需要 `launch.json`？

  

在沒有這個檔案的情況下，我們通常需要在終端機 (Terminal) 中手動輸入 `cargo run` 或 `cargo test`。

有了 `launch.json` 後，我們可以直接按下 **`F5`** 鍵啟動偵錯模式，支援設定中斷點 (Breakpoints)、逐步執行、查看變數內容等進階開發功能。

  

## 前置需求 (Prerequisites)

  

要讓這些設定正常運作，你需要：

1. **Rust 工具鏈**：已安裝 `rustup`、`cargo` 等。

2. **CodeLLDB 擴充套件**：VS Code/Cursor 的偵錯核心，用來驅動底層的 LLDB 偵錯器。

  

## 設定檔解析

  

目前的 `.vscode/launch.json` 中包含了兩個主要的設定 (Configurations)：

  

### 1. `Debug executable 'rust_tutorial'`

  

這個設定的用途相當於在終端機輸入 `cargo run`。

  

```json

{

    "name": "Debug executable 'rust_tutorial'",  // 顯示在偵錯面板選單上的名稱

    "type": "lldb",                              // 告訴編輯器使用 LLDB 偵錯器 (需安裝 CodeLLDB)

    "request": "launch",                         // 表示這是一個啟動程式的請求

    "cargo": {

        "args": [

            "run",

            "--bin=rust_tutorial"                // 告訴 cargo 要執行名稱為 rust_tutorial 的執行檔

        ]

    },

    "args": []                                   // 傳遞給你的程式的命令列參數 (如果你的程式需要接收參數可以寫在這裡)

}

```

  

- **使用時機**：當你想執行你的主要程式 (例如 `src/main.rs`) 並尋找 Bug 時。

  

#### 💡 補充：Rust 是如何知道程式的進入點的？（關於 `--bin` 的運作）

  

Rust（透過建置工具 **Cargo**）主要是透過**慣例大於設定 (Convention over Configuration)** 的方式來尋找專案進入點。

  

1. **檔案慣例 (`src/main.rs`)**

   當你建立一個新的專案，Cargo 發現 `src/main.rs` 存在時，它會自動將這個檔案視為一個二進位執行檔的預設進入點。這個檔案內必須有一個 `fn main()` 函式，這是程式開始執行的第一行位置。

  

2. **名稱來源 (`Cargo.toml`)**

   當我們在 `launch.json` 中寫 `--bin=rust_tutorial` 時，這個名稱來自於專案根目錄下的 `Cargo.toml` 檔案：

   ```toml

   [package]

   name = "rust_tutorial"

   ```

   因為 `name` 設定為 `"rust_tutorial"`，且存在 `src/main.rs`，所以 Cargo 自動推導出這裡有一個名為 `rust_tutorial` 的執行檔。

  

3. **可以有多個進入點嗎？**

   可以的！如果你的專案有多個可執行的工具，你可以建立 `src/bin/` 目錄。將其他的 `.rs` 檔案（例如 `src/bin/client.rs` 和 `src/bin/server.rs`）放入後，Cargo 會自動把它們視為獨立的執行檔（名稱分別為 `client` 和 `server`）。

   這時在 `launch.json` 中，只要把 `--bin` 改成你想要執行的名稱即可，例如 `--bin=client`。

  

### 2. `Debug unit tests in executable 'rust_tutorial'`

  

這個設定的用途相當於在終端機輸入 `cargo test`。

  

```json

{

    "name": "Debug unit tests in executable 'rust_tutorial'",

    "type": "lldb",

    "request": "launch",

    "cargo": {

        "args": [

            "test",                              // 執行測試

            "--bin=rust_tutorial"

        ]

    }

}

```

  

- **使用時機**：當你在程式碼中寫了 `#[test]` 並且想要單獨針對測試進行偵錯時。

  

## 如何使用？

  

1. 開啟 VS Code / Cursor 左側的 **執行與偵錯 (Run and Debug)** 面板 (或按下 `Ctrl+Shift+D` / `Cmd+Shift+D`)。

2. 在面板上方的下拉式選單中，你會看到上述設定的名稱。

3. 選擇你想執行的設定，然後點擊旁邊的綠色播放按鈕 (或按 `F5`)。

4. 程式啟動後，如果遇到你設定的中斷點 (在行號旁邊點擊出現的紅點)，程式就會暫停，讓你檢查當下的狀態。