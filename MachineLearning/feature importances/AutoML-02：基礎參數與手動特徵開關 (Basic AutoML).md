`demo_automl_basic.py`

```python
import numpy as np  
import pandas as pd  
import optuna  
import xgboost as xgb  
from sklearn.model_selection import train_test_split  
from sklearn.metrics import precision_score  
import warnings  
  
# 忽略不必要的警告，讓輸出畫面乾淨  
warnings.filterwarnings("ignore")  
  
  
# ==========================================  
# 1. 準備假資料 (Mock Data Generation)# ==========================================  
def generate_mock_data(n_samples=2000):  
    print(f"🔄 正在生成 {n_samples} 筆模擬量化交易特徵資料...")  
    np.random.seed(42)  
  
    # 建立基礎特徵 (模擬 Java 傳過來的 Base Data)    df = pd.DataFrame({  
        'volume_rate': np.random.uniform(0.5, 3.0, n_samples),  
        'flat_time_rate': np.random.uniform(0.0, 1.0, n_samples),  
        'rsi_14': np.random.uniform(20, 80, n_samples),  
        'macd_hist': np.random.normal(0, 1, n_samples)  
    })  
  
    # 隱藏的市場真實規則 (用來測試模型能不能把它找出來)  
    # 假設：交易量大 + RSI 低 + (持平時間率的 sin 值) 會導致高機率上漲  
    hidden_signal = (df['volume_rate'] * 1.5) - (df['rsi_14'] * 0.05) + np.sin(df['flat_time_rate'] * np.pi)  
    hidden_signal += np.random.normal(0, 1, n_samples)  # 加入市場雜訊  
  
    # 模擬三重屏障 (Triple Barrier) 的 PnL 結果  
    # top 15% 是大賺(1), bottom 20% 是大賠(-1), 中間 65% 是盤整(0)  
    df['pnl_label'] = 0  
    df.loc[hidden_signal > np.percentile(hidden_signal, 85), 'pnl_label'] = 1  
    df.loc[hidden_signal < np.percentile(hidden_signal, 20), 'pnl_label'] = -1  
  
    # 【做多模型 (LONG)】：我們只在乎能不能抓到 1，其他 0 和 -1 都視為「不要進場 (0)」  
    df['target_long'] = (df['pnl_label'] == 1).astype(int)  
  
    print(  
        f"📊 資料分佈: 不進場(0) = {(df['target_long'] == 0).sum()} 筆, 獲利進場(1) = {(df['target_long'] == 1).sum()} 筆")  
    return df  
  
  
df = generate_mock_data()  
  
  
# ==========================================  
# 2. 定義 Optuna 的尋寶邏輯 (Objective)# ==========================================  
def objective(trial):  
    """  
    Optuna 會不斷呼叫這個函數。  
    每次呼叫時，trial 會帶入一組全新的「隨機/猜測」參數。  
    """  
    # ----------------------------------------------------  
    # [A] 特徵工程自動化 (Automated Feature Engineering)    # ----------------------------------------------------    # 讓 Optuna 決定要不要套用 sin 函數 (True 或 False)    use_sin_transform = trial.suggest_categorical("use_sin_transform", [True, False])  
    # 讓 Optuna 尋找特徵相乘的最佳權重 (範圍 0.1 到 5.0)    vol_weight = trial.suggest_float("vol_weight", 0.1, 5.0)  
  
    X = df[['volume_rate', 'flat_time_rate', 'rsi_14', 'macd_hist']].copy()  
  
    # 根據 Optuna 給的參數，即時組合出新的特徵欄位！  
    if use_sin_transform:  
        X['magic_feature'] = (X['volume_rate'] * vol_weight) * np.sin(X['flat_time_rate'])  
    else:  
        X['magic_feature'] = (X['volume_rate'] * vol_weight) * X['flat_time_rate']  
  
    y = df['target_long']  
  
    # 切分訓練集與測試集 (80% 訓練, 20% 測試)  
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)  
  
    # ----------------------------------------------------  
    # [B] 模型超參數自動化 (Hyperparameter Optimization)    # ----------------------------------------------------    param = {  
        "n_estimators": trial.suggest_int("n_estimators", 50, 200),  
        "max_depth": trial.suggest_int("max_depth", 3, 7),  
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.2, log=True),  
        # 關鍵參數：對抗資料不平衡！讓演算法自己決定把稀有的「賺錢訊號」放大幾倍  
        "scale_pos_weight": trial.suggest_float("scale_pos_weight", 1.0, 10.0),  
        "random_state": 42,  
        "n_jobs": -1  
    }  
  
    # ----------------------------------------------------  
    # [C] 訓練與評估 (Training & Evaluation)    # ----------------------------------------------------    model = xgb.XGBClassifier(**param)  
    model.fit(X_train, y_train)  
  
    # 預測進場機率 (大於某個閥值才進場)  
    # 這裡我們連「進場信心閥值」都交給 Optuna 去尋找！(0.5 到 0.8 之間)  
    entry_threshold = trial.suggest_float("entry_threshold", 0.5, 0.8)  
  
    preds_prob = model.predict_proba(X_test)[:, 1]  
    preds_class = (preds_prob > entry_threshold).astype(int)  
  
    # 計算 Precision (精確率)  
    # 為什麼看 Precision？因為我們在乎的是「只要模型說會漲，就一定要漲！」  
    precision = precision_score(y_test, preds_class, zero_division=0)  
  
    # Optuna 會試圖「最大化」這個回傳值  
    return precision  
  
  
# ==========================================  
# 3. 啟動 AutoML 引擎  
# ==========================================  
if __name__ == "__main__":  
    print("\n🚀 開始 Optuna AutoML 尋寶之旅 (執行 50 輪測試)...")  
  
    # 告訴 Optuna 我們的目標是「最大化 (maximize)」 Precision    study = optuna.create_study(direction="maximize")  
  
    # 開始跑 50 輪 (可透過 n_trials 控制時間)  
    study.optimize(objective, n_trials=50)  
  
    # ==========================================  
    # 4. 印出尋寶結果  
    # ==========================================  
    print("\n" + "=" * 50)  
    print("🏆 AutoML 最佳結果出爐 🏆")  
    print("=" * 50)  
    print(f"🥇 測試集最高精確率 (Precision): {study.best_value:.4f}")  
    print("\n💡 讓模型達到最高勝率的「最佳參數組合」：")  
  
    for key, value in study.best_params.items():  
        if key in ['use_sin_transform', 'vol_weight']:  
            print(f"  [特徵發明] {key}: {value}")  
        else:  
            print(f"  [模型參數] {key}: {value}")  
  
    print("\n(您可以直接將這組參數抄寫回您的正式環境中使用了！)")
```

這是在導入 AutoML 初期最常見的作法。我們在 Python 程式碼中預先寫好幾種可能的特徵組合邏輯，並把「是否啟用」的控制權交給 Optuna。

### 核心觀念

- **手動定義搜索空間**：透過 `trial.suggest_categorical("use_sin_transform", [True, False])`，讓機器決定是否要對時間序列套用 `sin` 函數。
    
- **權重尋寶**：透過 `trial.suggest_float("vol_weight", 0.1, 5.0)` 尋找指標間的最佳乘數。
    
- **不平衡資料處理**：利用 `scale_pos_weight` 讓模型自動尋找放大稀有「賺錢訊號」的最佳倍率。
    

**極限與痛點**： 機器的聰明程度受限於「人類的想像力」。如果我們沒有在程式碼中寫下 `np.sin()`，機器就永遠不會去測三角函數。當特徵維度破千時，人類不可能手寫所有的排列組合。