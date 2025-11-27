
# **Git & GitHub 快速上手懶人包**

這是一份專為 Windows 使用者設計的 **Git & GitHub 快速上手懶人包**。

這份教學將解決你剛剛遇到的權限問題，並涵蓋日常開發最核心的「存檔、上傳、更新」流程。

---

### 第一步：安裝 GitHub CLI 工具 (一次性)

首先解決你電腦沒有 `gh` 指令的問題。

1. 開啟 Windows 搜尋，輸入 **PowerShell** 或 **CMD**。
    
2. 貼上以下指令並執行：
    
    
    
    ```PowerShell
    winget install --id GitHub.cli
    ```
    
    _(或者去 [GitHub CLI 官網](https://cli.github.com/) 下載 .msi 安裝檔一直按下一步)_
    
3. **重要：** 安裝完畢後，**關閉所有** 終端機視窗，重新開啟 Git Bash，輸入 `gh --version` 確認有跑出版本號。
    

---

### 第二步：登入 GitHub (解決權限問題)

這是最重要的一步，登入後 `git clone` 私有庫就不會再報錯了。

1. 在 Git Bash 輸入：
    
    
    
```Bash
   gh auth login
```
    
2. 鍵盤上下鍵選擇，按 Enter 確認：
    
    - `What account do you want to log into?` → 選 **GitHub.com**
        
    - `What is your preferred protocol?` → 選 **HTTPS**
        
    - `Authenticate Git with your GitHub credentials?` → 選 **Yes (Y)**
        
    - `How would you like to authenticate?` → 選 **Login with a web browser**
        
3. 終端機會顯示一組代碼 (例如 `1234-5678`)。
    
4. 按 **Enter**，瀏覽器會跳出，輸入該代碼並點擊 **Authorize** (授權)。
    
5. 回到終端機，顯示 `Logged in as ...` 即成功！
    

---

### 第三步：下載專案 (Clone)

現在你可以下載任何你有權限的 Repo (包含 Private 私有庫)：



```Bash
# 把雲端的程式碼抓下來
git clone https://github.com/kirinchendomi/test-test.git
```

---

### 第四步：日常開發流程 (寫扣、存檔、上傳)

這是你每天工作會重複 N 次的循環。請記住這三個步驟：**Add (加入) → Commit (確認存檔) → Push (推送到雲端)**。

假設你已經修改了一些程式碼...

**1. 進入專案資料夾 (超常忘記)**



```Bash
cd test-test
```

**2. 檢查狀態 (查看改了哪些檔案)**



```Bash
git status
# 紅色字：修改過但還沒準備存檔的檔案
# 綠色字：已經準備好要存檔的檔案
```

**3. 將檔案加入「暫存區」 (Add)**



```Bash
git add .
# "." 代表加入所有修改過的檔案
# 如果只想加特定檔案： git add main.py
```

4. 提交版本 (Commit)

這就像是遊戲的「存檔點」，必須寫備註 (Message)。



```Bash
git commit -m "這裡寫你改了什麼，例如：新增登入功能"
```

5. 推送到 GitHub (Push)

將你電腦的存檔，同步到 GitHub 網站上。



```Bash
git push
```

---

### 第五步：同步別人的進度 (Pull)

如果你換電腦工作，或是團隊有其他人更新了程式碼，你需要把它抓下來：



```Bash
git pull
```

---

### 💡 懶人指令速查表

|**動作**|**指令**|**白話文解釋**|
|---|---|---|
|**登入**|`gh auth login`|讓電腦擁有 GitHub 權限 (通常只需做一次)|
|**下載**|`git clone [網址]`|把整個專案從雲端複製一份下來|
|**檢查**|`git status`|我現在改了什麼？有沒有東西沒存？(迷路時專用)|
|**加入**|`git add .`|把桌上的文件收進信封袋 (準備寄出)|
|**存檔**|`git commit -m "..."`|把信封袋封口並蓋章 (正式確立一個版本)|
|**上傳**|`git push`|把信封寄出去 (更新到 GitHub 網站)|
|**更新**|`git pull`|把雲端最新的進度抓下來跟自己同步|

建議下一步：

試著照順序執行一次：修改一個檔案 -> git add . -> git commit -m "test" -> git push，然後去 GitHub 網頁看有沒有更新成功。如果成功，你就出師了！