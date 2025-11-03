好的，完全沒問題！這份教學將會基於您**已經有一個現成的 React Web 專案**（例如用 `create-react-app` 或 `Vite` 建立的專案），一步步教您如何使用 Capacitor 將它打包成 App，並在模擬器和實體手機上進行偵錯。

---

### Capacitor 實戰教學：將現有 React 專案打包為 iOS/Android App

本教學將帶您完成從整合 Capacitor、在本機模擬器運行，到最終在實體手機上偵錯的全過程。

#### 前置準備 (非常重要！)

在開始之前，請確保您的開發環境已準備就緒。Capacitor 雖然簡化了 App 開發，但它**不能取代**原生的開發工具鏈。

1. **一個現有的 React 專案**: 確保您的專案可以正常 `npm run build`。
    
2. **Node.js**: 確保已安裝 LTS 版本的 Node.js。
    
3. **Xcode**:
    
    - 如果您想開發 iOS App，**必須使用 macOS** 並從 App Store 安裝最新版的 Xcode。
        
    - 安裝後請至少打開一次，同意其條款並讓它完成元件安裝。
        
4. **Android Studio**:
    
    - 如果您想開發 Android App，請下載並安裝最新版的 Android Studio。
        
    - 安裝過程中，請確保勾選了 Android SDK、Platform-Tools 和一個模擬器映像檔。
        

---

### 第 1 步：在您的 React 專案中整合 Capacitor

1. **進入您的專案目錄**:
    
    
    
    ```Bash
    cd /path/to/your-react-project
    ```
    
2. **安裝 Capacitor 相關套件**:
    
    
    
    ```Bash
    npm install @capacitor/core @capacitor/cli
    npm install @capacitor/ios @capacitor/android
    ```
    
    - `@capacitor/core`: 核心 API。
        
    - `@capacitor/cli`: 命令列工具。
        
    - `@capacitor/ios`, `@capacitor/android`: iOS 和 Android 的平台支援。
        
3. **初始化 Capacitor**:
    
    
    
    ```Bash
    npx cap init
    ```
    
    CLI 會問您幾個問題：
    
    - `? App name?` (輸入您的 App 名稱，例如 `My Awesome App`)
        
    - `? App Package ID?` (輸入 App 的唯一 ID，慣例是反向網域名稱，例如 `com.company.myapp`)
        
    - `? Web asset directory?` (這是最關鍵的一步！)
        
        - 如果您的專案是用 **Create React App** 建立的，請輸入 `build`。
            
        - 如果您的專案是用 **Vite** 建立的，請輸入 `dist`。
            
    
    完成後，專案中會多一個 `capacitor.config.ts` (或 `.js`) 的設定檔。
    
4. **新增原生平台**:
    
    
    
    ```Bash
    npx cap add ios
    npx cap add android
    ```
    
    這個指令會在您的專案根目錄下建立 `ios` 和 `android` 資料夾。這兩個資料夾是**完整的、可以獨立運作的原生專案**。您的 React 程式碼未來會被複製到這裡面。
    

### 第 2 步：建置與同步您的 Web 程式碼

這是 Capacitor 的核心工作流程：**先建置 Web，再同步到原生專案**。

1. **建置您的 React 專案**:
    
    
    
    ```Bash
    npm run build
    ```
    
    這個指令會根據您的專案設定，在 `build` (或 `dist`) 資料夾中生成最新的靜態網站檔案。
    
2. **同步到原生平台**:
    
    
    
    ```Bash
    npx cap sync
    ```
    
    `sync` 指令會做兩件事：
    
    - 將您 `build` 資料夾中的所有內容複製到 `ios` 和 `android` 專案的正確位置。
        
    - 更新原生專案中的外掛程式碼。
        

**重點**: 每當您修改了 React 程式碼，都需要重複 `npm run build` -> `npx cap sync` 這兩個步驟來更新您的 App。

### 第 3 步：在模擬器 (Simulator/Emulator) 上運行

現在，讓我們在電腦上的虛擬手機裡看看成果。

#### 運行 iOS 模擬器

1. **在 Xcode 中打開專案**:
    
    
    
    ```Bash
    npx cap open ios
    ```
    
    這個指令會自動用 Xcode 打開 `ios/` 資料夾中的專案。
    
2. **選擇模擬器並運行**:
    
    - 在 Xcode 頂部的工具列，從下拉選單中選擇一個您喜歡的 iPhone 模擬器（例如 iPhone 14 Pro）。
        
    - 點擊左邊的 ▶️ (Run) 按鈕。
        
    
    Xcode 會開始建置 App 並啟動模擬器。稍等片刻，您就會看到您的 React 網站以 App 的形式運行起來了！
    

#### 運行 Android 模擬器

1. **在 Android Studio 中打開專案**:
    
    
    
    ```Bash
    npx cap open android
    ```
    
    這個指令會自動用 Android Studio 打開 `android/` 資料夾中的專案。
    
2. **等待 Gradle 同步**: 第一次打開時，Android Studio 需要一些時間下載依賴項並同步專案 (Gradle Sync)，請耐心等候。
    
3. **選擇模擬器並運行**:
    
    - 在 Android Studio 頂部的工具列，從下拉選單中選擇一個您已建立的虛擬裝置 (AVD)。如果沒有，可以從 `Tools > Device Manager` 中建立一個。
        
    - 點擊旁邊的 ▶️ (Run) 按鈕。
        
    
    Android Studio 會建置 App 並將其安裝到模擬器上。
    

### 第 4 步：偵錯 (Debug) - 找出問題的關鍵

這是最棒的部分。因為 App 核心是 WebView，所以您可以**直接使用瀏覽器的開發者工具**來進行偵錯！

#### 在 iOS 模擬器上偵錯 (使用 Safari)

1. **開啟 Safari 的開發者模式**:
    
    - 在您的 Mac 上打開 Safari 瀏覽器。
        
    - 前往 `Safari > 設定 (Preferences) > 進階 (Advanced)`。
        
    - 勾選最下方的 **「在選單列中顯示『開發』選單 (Show Develop menu in menu bar)」**。這是一次性設定。
        
2. **連接偵錯器**:
    
    - 確保您的 App 正在 iOS 模擬器上運行。
        
    - 回到 Mac 的 Safari，點擊頂部選單列的 `開發 (Develop)`。
        
    - 您會看到您的模擬器名稱，以及您 App 的名稱 (通常是 localhost)。
        
    - 點擊它，Safari 的 **Web 偵錯工具 (Web Inspector)** 就會彈出！
        
    
    現在，您可以在這裡看到 `console.log` 的輸出、檢查網路請求、檢視 DOM 元素、並在「來源 (Sources)」頁籤中為您的 JavaScript 程式碼**設定中斷點 (Breakpoints)**！
    

#### 在 Android 模擬器上偵錯 (使用 Chrome/Edge)

1. **打開 Chrome/Edge 遠端偵錯**:
    
    - 在您的電腦上打開 Chrome 或 Edge 瀏覽器。
        
    - 在網址列輸入 `chrome://inspect` 或 `edge://inspect` 並前往。
        
2. **連接偵錯器**:
    
    - 確保您的 App 正在 Android 模擬器上運行。
        
    - 在 `inspect` 頁面中，您應該會在 "Remote Target" 區塊看到您的模擬器和正在運行的 App。
        
    - 點擊您 App 下方的 **inspect** 連結。
        
    
    Chrome 開發者工具 (DevTools) 將會彈出，並已連接到您 App 的 WebView。所有您熟悉的偵錯工具，包括 `console`, `elements`, `network`, `breakpoints` 等，都可以在這裡使用。
    

### 第 5 步：在實體手機上運行與偵錯

流程與模擬器幾乎完全一樣，只是多了連接和信任設備的步驟。

#### 在實體 iPhone/iPad 上

1. **連接設備**: 使用 USB 線將您的 iPhone/iPad 連接到 Mac。
    
2. **信任電腦**: 在 iPhone/iPad 上解鎖，並在跳出的提示中點擊「信任」這部電腦。
    
3. **設定簽名**: 在 Xcode 中，點擊專案名稱，進入 `Signing & Capabilities` 頁籤。您需要選擇一個 Team (如果您沒有付費開發者帳號，可以使用您的個人 Apple ID 進行免費簽署)。
    
4. **運行**: 在 Xcode 頂部的裝置選單中，選擇您的實體手機，然後點擊 ▶️ (Run) 按鈕。
    
5. **偵錯**: **方法與模擬器完全相同**。打開 Mac 上的 Safari，`開發 (Develop)` 選單下會直接顯示您連接的實體手機名稱，點擊即可開始偵錯。
    

#### 在實體 Android 手機上

1. **開啟開發者模式**:
    
    - 到手機的 `設定 > 關於手機`。
        
    - 連續點擊「版本號碼 (Build number)」7 次，直到提示您已成為開發者。
        
2. **開啟 USB 偵錯**:
    
    - 回到 `設定 > 系統 > 開發人員選項`。
        
    - 找到並啟用「USB 偵錯 (USB debugging)」。
        
3. **連接設備**: 使用 USB 線將手機連接到電腦。在手機上跳出的視窗中，選擇「允許 USB 偵錯」。
    
4. **運行**: 在 Android Studio 頂部的裝置選單中，選擇您的實體手機，然後點擊 ▶️ (Run) 按鈕。
    
5. **偵錯**: **方法與模擬器完全相同**。打開電腦上的 Chrome/Edge，前往 `chrome://inspect`，您的實體手機和 App 將會出現在列表中，點擊 `inspect` 即可開始偵錯。
    

---

最佳實踐建議:

在開發過程中，90% 的 UI 開發和業務邏輯都可以在電腦瀏覽器 (npm start) 中完成。只有在需要測試 Capacitor 原生外掛 (如相機、GPS) 或檢查 App 在真實設備上的外觀時，才需要執行 build 和 sync 流程。這會大大提升您的開發效率！