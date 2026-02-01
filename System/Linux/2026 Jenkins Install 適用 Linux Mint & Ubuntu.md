這對 IT RD 來說是一個非常精確的問題。

 Linux Mint 的底層 **就是 Ubuntu LTS**（例如 Mint 21 是基於 Ubuntu 22.04 LTS），所以對於 Jenkins 這種 Server 級應用程式來說，二者是 99.9% 互通的。


### 1. 安裝過程的差異 (The "Codename" Trap)

這是唯一的差別，也是最容易讓腳本報錯的地方。

- **Ubuntu：** 使用 `lsb_release -cs` 指令會回傳 `jammy` (22.04) 或 `noble` (24.04)。
    
- **Linux Mint：** 使用 `lsb_release -cs` 會回傳 Mint 自己的代號，例如 `virginia` 或 `wilma`。
    

問題點：

如果你看網路上的教學（或是某些自動安裝腳本），它們通常會寫這樣：

Bash

```
# 錯誤示範 (在 Mint 上可能會失敗)
sudo add-apt-repository "deb https://pkg.jenkins.io/debian-stable $(lsb_release -cs) main"
```

這會失敗，因為 Jenkins 的官方儲存庫 (Repository) 裡沒有一個資料夾叫做 `virginia`，它只認得 Ubuntu 的代號 `jammy`。

解決方案 (正確做法)：

直接使用 Jenkins 官方文檔提供的 「通用寫法 (binary/)」，或者直接填寫對應的 Ubuntu 代號。

**Linux Mint 上的標準安裝 SOP (與 Ubuntu 通用)：**



```Bash
# 1. 安裝 Java (Jenkins 依賴)
sudo apt update
sudo apt install fontconfig openjdk-17-jre

# 2. 加入 Jenkins Key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

# 3. 加入 Repo (注意這裡！不要用變數，直接用 binary/ 最穩)
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# 4. 更新並安裝
sudo apt update
sudo apt install jenkins
```

_只要用這招，Mint 和 Ubuntu 安裝起來完全一模一樣。_

---

### 2. 使用上的差異 (完全無感)

安裝完後，**完全沒有差別**。

- **Service 管理**：都是用 Systemd。
    
    - 啟動：`sudo systemctl start jenkins`
        
    - 查看狀態：`sudo systemctl status jenkins`
        
    - Log路徑：`/var/log/jenkins/`
        
- **Web 介面**：都是瀏覽器打開 `localhost:8080`，介面長得一模一樣。
    
- **Pipeline 腳本**：
    
    - 在 Jenkins Pipeline 裡的 `sh '...'` 指令，因為 Mint 的 Shell 環境跟 Ubuntu 一樣都是 bash，且套件管理都是 apt，所以你寫的 CI/CD script **完全通用**。
        

---

### 3. 給 RD 的額外建議 (Mint 特有的注意事項)

雖然核心一樣，但 Mint 的預設環境配置跟 Ubuntu Server 版還是有點不同，建議你注意以下兩點：

A. 防火牆 (UFW) 預設是 GUI 管理

Ubuntu Server 通常沒開 GUI，但 Mint 有。如果你發現外部連不進 8080：

1. 去選單打開 **「防火牆組態 (Firewall)」**。
    
2. 新增規則 -> Simple -> Allow -> Port **8080**。
    
3. (或者用指令：`sudo ufw allow 8080`)
    

B. 記憶體監控 (針對 GPD MicroPC)

如果你是要裝在那台 GPD MicroPC 上：

- Jenkins 是出了名的「吃記憶體怪獸」 (Java Heap)。
    
- GPD 記憶體有限，建議修改 Jenkins 的啟動參數，限制它的記憶體用量。
    
- **修改方法**：
    
    Bash
    ```
    sudo nano /usr/lib/systemd/system/jenkins.service
    ```
    
    找到 `Environment="JAVA_OPTS=-Djava.awt.headless=true"` 這一行，在後面加上 `-Xmx1024m` (限制最大 1GB，數值看你硬體調整)：
    
    
    Bash
    ```
    Environment="JAVA_OPTS=-Djava.awt.headless=true -Xmx1024m"
    ```
    
    改完記得 `sudo systemctl daemon-reload` 和 `sudo systemctl restart jenkins`。
    

