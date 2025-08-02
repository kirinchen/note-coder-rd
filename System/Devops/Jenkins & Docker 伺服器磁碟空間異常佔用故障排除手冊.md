

### **伺服器磁碟空間異常佔用故障排除手冊 (Jenkins & Docker 環境)**

版本: 1.0

最後更新: 2025-08-01

#### **1. 概述**

本手冊旨在提供一套系統化的診斷與解決流程，用於處理 Linux 伺服器上因不明原因導致磁碟空間（特別是根目錄 `/`）被佔滿的問題。此流程特別適用於運行 Docker 和 Jenkins 的環境，因為這些服務的日誌和暫存檔案是常見的空間殺手。

#### **2. 故障症狀**

- 使用 `df -h` 指令檢查，發現根目錄 `/` 的 `Use%` 超過 90%。
    
- 執行標準的 Docker 清理指令 (`docker system prune -af`, `docker volume prune -f` 等) 後，空間並未顯著釋放。
    
- 直覺上懷疑是 Docker 或 Jenkins 的問題，但無法定位到具體的大檔案。
    

#### **3. 診斷流程 (由淺入深)**

##### **步驟 3.1: 初步評估 (檢查常見嫌疑犯)**

在進行全面掃描前，先快速檢查最常見的幾個問題點。

1. **檢查 Docker 自身空間評估**:
    
    Bash
    
    ```
    docker system df
    ```
    
    _觀察 `RECLAIMABLE` 欄位。如果此處空間很大，執行 `docker system prune -af` 應該有效。如果很小，代表問題不在 Docker 的閒置資源。_
    
2. **檢查 Jenkins 工作區 (Workspace)**:
    
    Bash
    
    ```
    sudo du -sh /var/lib/jenkins/workspace
    ```
    
    _CI/CD 過程中的原始碼、依賴套件會在此累積，是常見的元兇。_
    

##### **步驟 3.2: 全面盤查 (地毯式搜索)**

如果初步評估未找到元兇，代表問題隱藏在其他地方。我們需要從根目錄開始，找出系統中最大的目錄。

1. **從根目錄 `/` 開始掃描**:
    
    Bash
    
    ```
    sudo du -ah / --max-depth=1 | sort -hr
    ```
    
    _此指令會列出 `/` 下第一層所有目錄的大小，並由大到小排序。排在最頂部的通常就是問題所在。_
    
2. 逐層深入:
    
    假設上一步發現 /var 目錄最大，則繼續深入該目錄。
    
    Bash
    
    ```
    sudo du -ah /var --max-depth=1 | sort -hr
    ```
    
    _重複此過程，例如 `/var` -> `/var/log`，直到找到佔用數十 GB 的具體檔案或目錄。_
    

#### **4. 常見問題根源與解決方案**

##### **問題 A: 系統日誌檔失控 (`/var/log`)**

這是本次修復歷程的最終元兇，也是極其常見的問題。

- **現象**: `du` 掃描最終指向 `/var/log`，發現 `syslog`, `journal` 或其他日誌檔異常巨大。
    
- **立即處理 (治標)**:
    
    1. 刪除已輪替的舊日誌檔 (例如 `syslog.1`)：
        
        Bash
        
        ```
        sudo rm /var/log/syslog.1
        ```
        
    2. 清空當前正在寫入的日誌檔 (安全作法)：
        
        Bash
        
        ```
        sudo truncate -s 0 /var/log/syslog
        ```
        
- 根本解決 (治本):
    
    設定 logrotate 以自動管理日誌大小和存檔。
    
    1. 編輯設定檔：`sudo nano /etc/logrotate.d/rsyslog`
        
    2. 確保設定包含 `size` 限制，例如 `size 200M`，以防止日誌在一天內就爆掉。
        
        Ini, TOML
        
        ```
        /var/log/syslog
        {
                rotate 7
                daily
                size 200M  # 關鍵設定
                missingok
                notifempty
                compress
                ...
        }
        ```
        

##### **問題 B: Jenkins Agent 管理不當**

日誌檔失控通常只是症狀，背後的原因需要解決，例如失控的 Agent。

- **現象**:
    
    - `syslog` 中被特定服務的警告或錯誤訊息瘋狂洗版。
        
    - 使用 `ps -ef | grep agent.jar` 發現大量同名的 Java 進程殘留。
        
- 立即處理 (治標):
    
    使用 pkill 精準終止所有相關的殭屍進程。
    
    Bash
    
    ```
    sudo pkill -f "agent.jar"
    ```
    
    **警告**: 絕對不要使用 `killall java`，這會殺掉伺服器上所有 Java 應用。
    
- 根本解決 (治本):
    
    使用 Systemd 來守護 Agent 進程。這是確保 Agent 穩定運行的最佳實踐，能實現自動重啟、日誌管理和標準化控制。
    
    1. 建立服務檔案 `sudo nano /etc/systemd/system/jenkins-agent.service`。
        
    2. 填入以下模板，確保 `User` 和 `WorkingDirectory` 正確：
        
        Ini, TOML
        
        ```
        [Unit]
        Description=Jenkins Agent Service
        After=network.target
        
        [Service]
        User=jenkins
        WorkingDirectory=/home/jenkins/agent
        ExecStart=java -jar agent.jar [所有參數...]
        Restart=always
        RestartSec=10
        
        [Install]
        WantedBy=multi-user.target
        ```
        
    3. 使用 `sudo systemctl start|stop|status|enable jenkins-agent` 進行管理。
        

#### **5. 總結與最佳實踐**

1. **從大處著手**: 當磁碟滿時，不要只盯著你懷疑的服務，用 `du --max-depth=1` 從根目錄開始掃描。
    
2. **區分症狀與原因**: 磁碟佔滿是「症狀」，日誌檔巨大是「直接原因」，而應用程式洗版日誌才是「根本原因」。要一併處理。
    
3. **治標更要治本**: 清理檔案只能暫時緩解，配置 `logrotate` 和使用 `Systemd` 才能一勞永逸。
    
4. **現代化工具**: 使用 `Systemd` 管理服務，它比傳統的 `nohup` 和手動維護的 PID 檔案更穩定、更強大。
    
5. **定期巡檢**: 定期使用 `ps` 檢查是否有異常的殭屍進程，並留意系統日誌。