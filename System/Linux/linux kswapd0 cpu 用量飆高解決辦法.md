

1. **兇手抓到了：`kswapd0` (PID 44)**
    
    - **CPU 48.4%**：`kswapd0` 是 Linux 核心負責「回收記憶體」的程序。當它佔用這麼高 CPU 時，代表實體記憶體嚴重不足，系統正瘋狂地試圖把資料搬去 Swap（虛擬記憶體）來清出空間。
        
    - **結果**：CPU 的 **`97.0 sy` (System usage)** 證實了這一點。您的 CPU 有 97% 的時間都在處理核心雜務（記憶體管理），只有 **3.0%** 的時間在跑您的應用程式。
        
2. **記憶體與 Swap 全部爆滿**
    
    - **RAM**: 1968.0 total, **53.9 free** (幾乎為 0)。
        
    - **Swap**: 512.0 total, **511.8 used** (100% 滿)。
        
    - **Avail Mem**: **1.5 avail** (這是最危險的數字，系統完全沒有餘裕，下一個新請求進來就會觸發 OOM Killer 殺掉程式)。
        
3. **Load Average 異常飆高**
    
    - Load Average: **43.28**。
        
    - 對於一台 2GB RAM 的機器（通常是 1核或 2核），Load 超過 2~4 就已經很慢了。43 代表有 40 幾個程序在排隊等待 CPU，但 CPU 正被 `kswapd0` 綁架中。
        

---


### 🛡️ 長期解法 (治本)



#### 1. 增加 Swap (最推薦，省錢且有效)

您的 Swap 只有 512MB，太小了。對於 2GB 的機器，建議給它 **2GB ~ 4GB** 的 Swap 緩衝。這不會讓速度變快，但能**防止當機**。

Bash

```
# 1. 關閉現有 swap
sudo swapoff -a

# 2. 建立一個 4GB 的 swap file
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096
sudo chmod 600 /swapfile
sudo mkswap /swapfile

# 3. 啟用
sudo swapon /swapfile

# 4. 寫入 fstab 確保重開機還在
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### 2. 限制 Docker 記憶體

防止某個容器（例如 MLflow 或 Jenkins）把整台機器拖垮。在 `docker-compose.yml` 中加上限制：

YAML

```
services:
  mlflow:
    # ...
    deploy:
      resources:
        limits:
          memory: 400M  # 限制 MLflow 最多吃 400MB
  
  db:
    # ...
    deploy:
      resources:
        limits:
          memory: 300M
```

#### 3. 調整 Java Heap (針對 Jenkins)

如果您一定要在這台跑 Jenkins，請務必限制它的 Java 記憶體上限 (Xmx)，否則它會預設吃掉大部分 RAM。

- 在 Jenkins 的 `JAVA_OPTS` 加入 `-Xmx512m`。
    

