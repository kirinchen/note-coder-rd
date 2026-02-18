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
