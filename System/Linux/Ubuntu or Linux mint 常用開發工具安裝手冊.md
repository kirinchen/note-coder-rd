## 安裝 Postman
在 Linux Mint 上，安裝 Postman 最推薦的方式是使用 **Flatpak**。因為 Postman 經常更新，用 Flatpak 可以自動跟隨最新版，且不會把你的系統函式庫弄髒。

**方法一：使用 Flatpak (推薦)**

Linux Mint 預設就支援 Flatpak，直接一行指令搞定：



```Bash
flatpak install flathub com.getpostman.Postman
```

安裝完後，直接在選單搜尋 "Postman" 即可開啟。

**方法二：手動下載 Tarball (傳統 Linux 作法)**

如果你不喜歡 Flatpak 的沙盒機制，想要最原始的二進位檔：

1. 去 [Postman 官網](https://www.postman.com/downloads/) 下載 Linux 64-bit 的 `.tar.gz`。
    
2. 解壓縮到 `/opt` (標準第三方軟體目錄)：
    
    
    
    ```Bash
    cd Downloads
    sudo tar -xzf postman-linux-x64.tar.gz -C /opt
    ```
    
3. 建立軟連結 (Symlink) 讓你能直接在終端機打 `postman`：
    
    
    
    ```Bash
    sudo ln -s /opt/Postman/Postman /usr/bin/postman
    ```
    
4. 建立桌面圖示 (Optional)：
    
    你需要手動寫一個 `.desktop` 檔案放到 `~/.local/share/applications/`，比較麻煩。所以我建議直接用 **方法一**。
    

---


# Linux Mint 安裝 IntelliJ IDEA 的最佳實踐：避開 FUSE2 依賴與路徑陷阱

**作者：Kirin Chen**

**環境：Linux Mint (MATE/Cinnamon) / High-Performance Backtesting Machine**

## 前言

對於 Java/Python 開發者來說，IntelliJ IDEA 是最強大的 IDE。雖然 Linux Mint 的軟體管理員 (Software Manager) 提供了一鍵安裝的 Flatpak 版本，但對於需要 **多版本管理 (EAP vs Stable)**、**高性能回測 (Backtesting)** 以及 **避免沙盒權限問題** 的資深 RD 來說，使用 **JetBrains Toolbox** 才是最正規的安裝方式。

這篇文章記錄了在 Linux Mint 上安裝 Toolbox 的完整步驟，並解決了常見的 `libfuse2` 缺失與路徑錯誤問題。

---

## Part 1: 為何不推薦 Flatpak / Snap？

雖然 `apt install` 或軟體中心很方便，但在開發環境中它們有兩個致命傷：

1. **沙盒限制 (Sandbox)**：Flatpak 版本的 IDE 運行在沙盒中，抓取系統 JDK (`/usr/lib/jvm`) 或呼叫外部工具 (Docker, Git) 時常遇到路徑對應問題。
    
2. **更新延遲**：無法第一時間取得 JetBrains 的 Hotfix 或 EAP (Early Access Program) 版本。
    

因此，官方推薦的 **JetBrains Toolbox** (AppImage 機制) 是最佳解。

---

## Part 2: 安裝步驟 (Terminal 流)

### 1. 關鍵依賴：安裝 libfuse2

這是 Linux Mint 21+ (基於 Ubuntu 22.04) 使用者最容易踩到的坑。新版 Linux 預設使用 FUSE3，但 JetBrains Toolbox 目前仍依賴 FUSE2。如果不裝這個，點擊執行檔會完全沒反應。



```Bash
sudo apt update && sudo apt install -y libfuse2
```

### 2. 下載與解壓縮

我們建立一個專門的目錄 (`~/apps`) 來管理它，保持家目錄乾淨。



```Bash
# 1. 建立目錄
mkdir -p ~/apps/toolbox

# 2. 下載 Toolbox (使用 wget)
wget -c "https://data.services.jetbrains.com/products/download?code=TBA&platform=linux" -O jetbrains-toolbox.tar.gz

# 3. 解壓縮
# --strip-components=1 會把解壓出來的第一層資料夾去掉，直接把內容物放進 ~/apps/toolbox
tar -xzf jetbrains-toolbox.tar.gz -C ~/apps/toolbox --strip-components=1
```

### 3. 首次啟動 (注意路徑陷阱！)

很多教學會叫你直接執行 `jetbrains-toolbox`，但其實執行檔藏在 `bin` 資料夾裡。



```Bash
# 進入 bin 目錄
cd ~/apps/toolbox/bin

# 執行 (注意前面的 ./)
./jetbrains-toolbox
```

_啟動後，Toolbox 會出現在系統匣 (Tray)，並自動將捷徑加入開始選單。以後你就不需要再打指令了。_
