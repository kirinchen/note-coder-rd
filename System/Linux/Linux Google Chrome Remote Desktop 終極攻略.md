
# [Linux 實戰] Google Chrome Remote Desktop 終極攻略：從 Headless 安裝到解決「Session 撞名」慘案

**作者：Kirin Chen**

**環境：Linux Mint (MATE) / GPD MicroPC**

## 前言

對於 IT RD 來說，Google Chrome Remote Desktop (CRD) 是最方便的遠端方案：不用打洞、不用搞 VPN，只要有 Google 帳號就能穿透內網連線。但在 Linux (特別是 Mint/Ubuntu) 上，它並不像 Windows 那樣「裝好就能用」，特別是 **「Session 撞名 (Could not acquire name on session bus)」** 這個錯誤，足以讓許多人卡關一整天。

這篇文章記錄了正確的安裝流程，以及如何優雅解決 Session 衝突問題。

---

## Part 1: 正確的安裝姿勢 (Command Line 流)

在 Linux 上，不要依賴 Chrome 瀏覽器介面上的「安裝」按鈕，直接走 **Headless (指令模式)** 設定最穩。

### 1. 下載與安裝 Host 程式

首先，去官網下載 `.deb` 包並安裝，或直接用指令：



```Bash
# 下載
wget https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb

# 安裝 (如果報錯依賴問題，系統會自動提示修復，或執行 apt --fix-broken install)
sudo apt install ./chrome-remote-desktop_current_amd64.deb
```

### 2. 建立設定目錄 (90% 的人都死在這裡)

Linux 版的 CRD 有個長年的 Bug：如果設定目錄不存在，它不會自己建，而是直接報錯。**請務必手動建立：**



```Bash
mkdir -p ~/.config/chrome-remote-desktop
```

### 3. 取得授權碼並註冊

1. 在本機 (或另一台電腦) 開啟 Chrome，前往：[remotedesktop.google.com/access](https://remotedesktop.google.com/access)
    
2. 選擇 **「透過 SSH 設定 (Set up via SSH)」** -> **「開始」** -> **「授權」**。
    
3. 複製網頁上產生的那串 `DISPLAY= /opt/...` 指令。
    
4. 回到 Linux Mint 終端機貼上執行，並設定 6 位數 PIN 碼。
    

---

## Part 2: 災難現場 —— 「雙胞胎撞名」慘案

當你興高采烈地設定好，重開機想登入時，卻發現本機螢幕跳出一個白色錯誤視窗，寫著：

> **Could not acquire name on session bus**

然後你就被踢出登入畫面，無限迴圈，連桌面都進不去。

### 發生原因 (Root Cause Analysis)

這是一個經典的 **D-Bus Session Conflict**。

Linux 的桌面環境 (Desktop Environment, DE) 通常假設「一個使用者在同一個時間點，只能擁有一個主要的圖形 Session」。

1. **CRD 的行為**：CRD 服務設定為開機自啟，它在背景偷偷幫你啟動了一個 Session (預設可能是嘗試啟動 MATE/GNOME)。
    
2. **本機的行為**：當你嘗試從物理螢幕 (GPD MicroPC) 登入時，你的本機 Session 也試圖註冊同樣的 D-Bus 名稱。
    
3. **結果**：系統判定名稱衝突 (Name already acquired)，於是後進者 (本機) 被踢出，或者兩邊都卡死。
    
### RD 的長久解法：如何共存？

如果你希望「平常可以用本機，出門可以用遠端」，而不要每次都撞車，你有兩個選擇：

**解法 A：手排模式 (最推薦 IT 人)**

- **平常**：CRD 服務保持關閉 (Stop)，本機快樂使用。
    
- **出門前**：SSH 進去 Linux，輸入 

```bash
  /opt/google/chrome-remote-desktop/chrome-remote-desktop --start
```
  
  這時候遠端會通，但本機螢幕可能會被鎖住或登出（正常現象）。