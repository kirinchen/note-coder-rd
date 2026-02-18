這是一份升級版的 **VS Code Git 全攻略**，從最基礎的環境配置 (Config) 到進階操作一次收錄。這篇文章適合剛安裝好 VS Code，準備將它打造成最強版本控制指揮中心的開發者。

---

# VS Code Git 全攻略：從環境設定到進階版控 (2025 最新版)

很多開發者習慣用終端機 (Terminal) 下 Git 指令，覺得這樣才「硬派」。但其實 VS Code 內建的 Git GUI 已經強大到能處理 95% 的日常需求，特別是在 **解決衝突 (Merge Conflicts)** 和 **檢視變更 (Diff)** 時，圖形介面有著絕對優勢。

這篇指南將帶你從零開始，把 VS Code 設定成最強大的 Git 工具。

---

## 第一步：環境配置 (The Foundation)

在打開 VS Code 之前，我們得先確保底層的 Git 設定是正確的。這一 步不做，後面會一直報錯。

### 1. 安裝與身份設定

打開你的終端機 (Terminal) 或 PowerShell，輸入以下指令告訴 Git 你是誰：



```Bash
# 設定你的名字
git config --global user.name "你的名字"

# 設定你的 Email (最好跟 GitHub/GitLab 帳號一致)
git config --global user.email "your.email@example.com"
```


## 步驟 1.5：取得專案 (Git Clone)

如果你是加入一個已經存在的團隊，或者想從 GitHub 下載開源專案，你不需要 `git init`，而是需要 `git clone`。在 VS Code 中，這件事變得超級簡單，連指令都不用打。

### 方法一：使用命令面板 (最推薦，不用滑鼠)

這是 VS Code 的精隨操作，快速又帥氣。

1. **複製網址**：先去 GitHub/GitLab 專案頁面，點擊綠色的 `Code` 按鈕，複製 `.git` 結尾的網址 (HTTPS 或 SSH 皆可)。
    
2. **呼叫指令**：在 VS Code 中按下 **`F1`** 或 **`Ctrl + Shift + P`** (Mac: `Cmd + Shift + P`) 開啟命令面板。
    
3. **輸入 Clone**：輸入 `Git: Clone` 並按下 Enter。
    
4. **貼上網址**：將剛剛複製的網址貼上，再按 Enter。
    
5. **選擇存放位置**：VS Code 會跳出檔案總管，請選擇一個你想要存放專案的 **「資料夾」** (例如 `D:\Projects`)。
    
6. **立刻開啟**：下載完成後，右下角會跳出通知：「Would you like to open the cloned repository?」，直接點擊 **Open** (開啟) 或 **Open in New Window** (在新視窗開啟)。
    

### 方法二：使用「來源控制」面板 (視覺化操作)

如果你剛打開 VS Code，什麼檔案都沒開，這是最直覺的方式。

1. 點擊左側的 **「來源控制」 (Source Control)** 圖示 (那個分岔的樹枝圖案)。
    
2. 你會看到兩個大按鈕：`Open Folder` 和 `Clone Repository`。
    
3. 點擊 **Clone Repository**。
    
4. 接著步驟同上，貼上網址並選擇存放路徑即可。
    

> **小撇步 (Pro Tip)**：
> 
> VS Code 甚至支援直接貼上 GitHub 的 **網頁網址** (不一定要 `.git` 結尾)，它會自動幫你辨識並下載！

---

### 這樣你的 Blog 結構就會更完整：

1. **環境配置** (安裝 Git, 設定 Name/Email)
    
2. **取得專案 (Git Clone)** <--- _插入這裡_
    
3. **介面導覽** (Stage, Commit, Push/Pull)
    
4. **分支操作** (Branching)
    
5. **進階功能** (解決衝突)
    
6. **必裝插件** (GitLens, Git Graph)

### 2. 關鍵設定：讓 VS Code 成為 Git 的預設編輯器

這是許多人忽略的一步！設定後，當你執行 `git rebase` 或 `git commit` (沒帶訊息) 時，Git 會自動打開 VS Code 讓你編輯，而不是把你卡在難用的 Vim 裡。



```Bash
git config --global core.editor "code --wait"
```

### 3. VS Code 內部設定最佳化

打開 VS Code，按下 `Ctrl + ,` (Windows) 或 `Cmd + ,` (Mac) 進入設定，搜尋並調整以下項目：

- **Git: Autofetch** -> **開啟 (True)**
    
    - _作用_：VS Code 會自動幫你從遠端抓取更新 (fetch)，讓你隨時看到落後幾個版本，不用手動按。
        
- **Git: Confirm Sync** -> **取消勾選 (False)** (選用)
    
    - _作用_：如果你討厭每次 Push 都要跳出確認視窗，可以關掉它。但新手建議保留以防手滑。
        
- **Git: Enable Smart Commit** -> **開啟 (True)**
    
    - _作用_：如果你沒有暫存 (Stage) 任何檔案就按 Commit，它會自動幫你把「所有變更」都 Commit 上去，省去一個步驟。
        

---

## 第二步：介面導覽與基礎操作

VS Code 的 Git 功能主要集中在左側的 **「來源控制」(Source Control)** 面板（圖示看起來像分岔的樹枝，快捷鍵 `Ctrl + Shift + G`）。

### 1. 初始化專案 (Git Init)

如果你的資料夾還不是 Git 倉庫，打開面板後點擊 **Initialize Repository** 即可瞬間建立。

### 2. 暫存與提交 (Stage & Commit)

這是最頻繁的操作循環：

- **檢視變更**：點擊檔案名稱，右側視窗會顯示 **Diff (差異比對)**，左邊是舊的，右邊是新的，修改處一目了然。
    
- **暫存 (Stage)**：點擊檔案旁邊的 `+` 號（相當於 `git add`）。
    
- **提交 (Commit)**：在上方輸入框打好訊息，按 `Ctrl + Enter`（相當於 `git commit`）。
    

### 3. 同步 (Push & Pull)

左下角的狀態列會有一個循環圖示。

- **箭頭向上**：你有幾個 Commit 還沒推送到遠端。
    
- **箭頭向下**：遠端有幾個更新你還沒拉下來。
    
- **點擊圖示**：執行 `git pull` 然後 `git push` (Sync)。
    

---

## 第三步：分支操作 (Branching)

別再用指令切分支了，VS Code 左下角就是神器。

- **建立/切換分支**：點擊視窗左下角的 **分支名稱** (例如 `main` 或 `master`)，上方會跳出選單。你可以直接選其他分支切換，或是選擇 `+ Create new branch...`。
    
- **刪除分支**：按 `F1` 輸入 `Git: Delete Branch`，選擇要刪除的分支。
    

---

## 第四步：進階功能 - 解決衝突 (Merge Conflicts)

這是 VS Code 最強大的功能。當 `git merge` 發生衝突時，VS Code 會自動幫你標記出來，並提供四個按鈕：

1. **Accept Current Change**：保留你目前的修改（綠色區塊）。
    
2. **Accept Incoming Change**：採用遠端傳入的修改（藍色區塊）。
    
3. **Accept Both Changes**：兩個都保留。
    
4. **Compare Changes**：開啟並排視窗詳細比較。
    

**Pro Tip**: 解決完所有衝突後，把檔案加入暫存區 (Stage)，VS Code 就知道你解決完了。

---

## 第五步：必裝神級插件 (Extensions)

VS Code 原生功能已經不錯，但加上這兩個插件才算完全體：

### 1. Git Graph

- **功能**：提供一個色彩繽紛的 **地鐵路線圖**，讓你視覺化看到所有分支的走向、合併點和 Commit 歷史。
    
- **用法**：安裝後，來源控制面板上方會多一個小圖示，點擊即可查看漂亮的線圖。在此圖表上對任意 Commit 按右鍵，還可以直接執行 `Cherry-pick` 或 `Revert`。
    

### 2. GitLens (由 GitKraken 開發)

- **功能**：最強大的 Git 增強工具。
    
- **殺手級功能**：**Current Line Blame**。只要你的游標點到某一行程式碼，它會在後面用淡淡的灰色字顯示：「這行是誰、在多久以前寫的、Commit 訊息是什麼」。抓戰犯（或回憶自己寫了什麼）超好用！
