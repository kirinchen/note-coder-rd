# Poetry 版 MLflow + XGBoost Hello World 指南


沒問題！使用 **Poetry** 來管理依賴絕對是更好的選擇，它能確保你的環境版本鎖定（Locking），讓未來換電腦或部署到 Server 時不會炸開。

這是一份更新後的 **Poetry 版 MLflow + XGBoost Hello World 指南**。

---

### Step 1: 專案初始化與 Poetry 設定

假設你已經安裝好 Poetry (如果沒裝：`pip install poetry`)。

**1. 建立專案資料夾**



```Bash
mkdir mlflow-poetry-starter
cd mlflow-poetry-starter
```

**2. 初始化 Poetry 環境**

這裡我們直接用指令快速建立 `pyproject.toml` 並安裝需要的套件。



```Bash
# 初始化 poetry (使用 -n 略過互動問答，直接用預設值)
poetry init -n --name "mlflow-starter" --description "A Hello World example for MLflow with XGBoost"

# 一次安裝所有依賴
poetry add mlflow xgboost pandas scikit-learn
```

此時你的目錄會多出 `pyproject.toml` 和 `poetry.lock`。

---

### Step 2: 撰寫 `docker-compose.yml`

維持與之前一樣的設定，負責啟動 MLflow Server。



```YAML
version: '3.7'

services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.10.0
    container_name: mlflow_server
    ports:
      - "5000:5000"
    volumes:
      - ./mlflow_data:/mlflow
    command: >
      mlflow server
      --backend-store-uri sqlite:////mlflow/mlflow.db
      --default-artifact-root /mlflow/artifacts
      --host 0.0.0.0
      --port 5000
```

---

### Step 3: 撰寫訓練程式 `train.py`

程式碼邏輯不變，但我們可以加上一點註解，讓你清楚這是在 Poetry 環境下執行的。



```Python
import mlflow
import mlflow.xgboost
import xgboost as xgb
from sklearn.datasets import load_diabetes
from sklearn.model_selection import train_test_split
from sklearn.metrics import r2_score

# ==========================================
# 1. 設定 MLflow 連線
# ==========================================
# 指向 Docker 啟動的 Server
mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("Poetry_XGBoost_Demo")

def train():
    # ==========================================
    # 2. 準備資料
    # ==========================================
    X, y = load_diabetes(return_X_y=True, as_frame=True)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    # ==========================================
    # 3. 啟動 MLflow Run
    # ==========================================
    with mlflow.start_run():
        print("🚀 [Poetry Env] 開始訓練...")
        
        # 開啟 XGBoost 自動紀錄
        mlflow.xgboost.autolog()

        # ==========================================
        # 4. 訓練 XGBoost 模型
        # ==========================================
        params = {
            "n_estimators": 100,
            "max_depth": 3,
            "learning_rate": 0.1,
            "subsample": 0.8,
            "early_stopping_rounds": 10
        }
        
        model = xgb.XGBRegressor(**params)
        
        model.fit(
            X_train, y_train, 
            eval_set=[(X_test, y_test)], 
            verbose=False
        )

        # ==========================================
        # 5. 手動補紀錄 Metric
        # ==========================================
        predictions = model.predict(X_test)
        r2 = r2_score(y_test, predictions)
        mlflow.log_metric("test_r2_score", r2)
        
        print(f"✅ 訓練完成！ R2 Score: {r2:.4f}")
        print("👉 請打開瀏覽器查看: http://localhost:5000")

if __name__ == "__main__":
    train()
```

---

### Step 4: 執行與驗證

這是使用 Poetry 最關鍵的一步，所有的執行都要透過 `poetry run`，或是進入 `poetry shell`。

**1. 啟動 MLflow Server (Docker)**



```Bash
docker-compose up -d
```

**2. 執行訓練 (透過 Poetry)**



```Bash
poetry run python train.py
```

_這行指令會確保 python 是使用 Poetry 建立的虛擬環境，而不是你系統原本的 python。_

**3. 查看結果**

打開瀏覽器： `http://localhost:5000`

---

## 可能會遇到的問題

### **MLflow** 版本（3.9.0）還不支援 **Pandas 3.0.0** 以上的版本

執行時遇到
```Plaintext
Because no versions of mlflow match >3.9.0,<4.0.0

 and mlflow (3.9.0) depends on pandas (<3), mlflow (>=3.9.0,<4.0.0) requires pandas (<3).

So, because mlflow-docker-simple depends on both mlflow (^3.9.0) and pandas (^3.0.0), version solving
```

#### 解決方法

你只需要在安裝時，**明確限制 Pandas 的版本在 3.0.0 以下**。

請在終端機執行以下指令：
```bash
poetry add mlflow "pandas<3.0.0" xgboost scikit-learn
```

### XGBoost 下載太久

[[Poetry 和 xgboost  疑難雜症#XGBoost 檔案太大下載太久]]