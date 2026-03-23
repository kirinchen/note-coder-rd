
# 實戰筆記：從零打造 Production-Ready 的 MLflow Server (踩坑全紀錄)

在 MLOps 的路這條路上，MLflow 幾乎是標配。但從「在本機隨便跑個 `mlflow ui`」進階到「在雲端 Server 用 Docker 部署並多人協作」，中間其實藏著不少細節與坑。

這篇文章整理了我在將 MLflow 部署到 Linode Server (使用 Docker Compose, S3, PostgreSQL) 過程中遇到的 5 個關鍵問題與解決方案。如果你也正卡在權限驗證或連線錯誤，這篇筆記或許能救你一命。

---

## 架構目標

我們的目標是建立一個**完全自包含 (Self-contained)** 的 MLflow 環境：

- **Tracking Server**: 使用 Docker 容器運行。
    
- **Backend Store**: 使用 PostgreSQL (存參數、指標)。
    
- **Artifact Store**: 使用 S3 兼容存儲 (存模型檔案)。
    
- **Security**: 開啟 Basic Auth (帳號密碼登入) 並限制外網存取。
    

---

## 踩坑 1：依賴地獄 - Pandas 與 MLflow 的愛恨情仇

**問題描述：**

在使用 Poetry 安裝環境時，發現最新的 `mlflow (3.9.0)` 與最新的 `pandas (3.x)` 發生衝突，導致安裝失敗。

**錯誤訊息：**



```Plaintext
Because mlflow (3.9.0) depends on pandas (<3), mlflow requires pandas (<3).
```

**解決方案：**

MLflow 3.9.0 尚未完全支援 Pandas 3.0.0 的 Copy-on-Write 新特性。解法是在 `pyproject.toml` 或安裝指令中明確鎖定 Pandas 版本：



```Bash
poetry add mlflow "pandas<3.0.0"
```

此外，如果下載 `xgboost` 等大檔很慢，強烈建議將 Poetry 的來源切換到台灣國網中心 (NCHC)：



```Bash
poetry source add --priority=primary nchc https://free.nchc.org.tw/pypi/simple
```

---

## 踩坑 2：Client 與 Server 的代溝 (404 Error)

**問題描述：**

Client 端使用最新的 MLflow 3.x，但 Docker Server 端使用舊版的 Image (`v2.10.0`)。訓練時雖然參數有紀錄，但在上傳模型時報錯。

**錯誤訊息：**



```Plaintext
API request to endpoint /api/2.0/mlflow/logged-models failed with error code 404
```

**原因與解法：**

新版 Client 呼叫了舊版 Server 不支援的 API。

**解法：** 確保 Docker Image 使用最新版。



```YAML
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    # ...
```

---

## 踩坑 3：部署到雲端後的 "Invalid Host Header"

**問題描述：**

在本機 `localhost` 跑得好好的，一部署到 Linode Server 用 IP 存取，瀏覽器卻顯示 `403 Forbidden` 或 `Invalid Host Header`。

**原因：**

這是 MLflow 新版的安全性機制（DNS Rebinding Protection），預設只信任 `localhost`。當請求的 Host Header 是 `139.162.x.x` 時，會被拒絕。

**解決方案：**

在 `docker-compose.yml` 的啟動指令中，必須同時加上 `--host 0.0.0.0` (監聽所有介面) 與 `--allowed-hosts "*"` (允許所有域名)。



```YAML
command: >
  mlflow server
  --host 0.0.0.0
  --allowed-hosts "*"
  # ... 其他參數
```

---

## 踩坑 4：開啟帳號密碼驗證 (Basic Auth) 的層層關卡

為了不讓 MLflow 在網路上裸奔，我們決定開啟 `--app-name basic-auth`。結果引發了一連串錯誤：

1. **缺少套件**：官方 Image 為了輕量化，預設沒裝驗證套件。
    
    - **錯誤**：`ModuleNotFoundError: No module named 'flask_wtf'`
        
    - **解法**：建立自定義 `Dockerfile`：
        
        
        
        ```Dockerfile
        FROM ghcr.io/mlflow/mlflow:latest
        RUN pip install psycopg2-binary "mlflow[auth]"
        ```
        
2. **缺少 Secret Key**：CSRF 防護需要金鑰。
    
    - **錯誤**：`A static secret key needs to be set for CSRF protection.`
        
    - **解法**：在 `docker-compose` 加入環境變數 `MLFLOW_FLASK_SERVER_SECRET_KEY`。
        
3. **初始帳號設定**：如何自動化建立 Admin？
    
    - **解法**：掛載 `basic_auth.ini` 設定檔。
        

---

