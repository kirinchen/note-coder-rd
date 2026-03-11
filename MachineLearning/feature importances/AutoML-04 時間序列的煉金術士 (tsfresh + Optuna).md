`demo_automl_tsfresh.py`

```python
import pandas as pd  
import numpy as np  
import optuna  
import xgboost as xgb  
from sklearn.model_selection import train_test_split  
from sklearn.metrics import precision_score  
from tsfresh import extract_features, select_features  
from tsfresh.utilities.dataframe_functions import impute  
import warnings  
  
warnings.filterwarnings("ignore")  
  
  
# ==========================================  
# 1. 準備時間序列資料 (修正版：確保有 1 和 0)# ==========================================  
def generate_ts_data(n_samples=300, n_steps=16):  
    print(f"🔄 正在生成 {n_samples} 筆交易機會 (每筆包含 {n_steps} 個時間序列 Section)...")  
    np.random.seed(42)  
  
    data_list = []  
    hidden_signals = []  # 用來收集所有的訊號分數  
  
    # 第一階段：產生特徵與原始訊號分數  
    for run_id in range(n_samples):  
        sdWidth_series = np.random.uniform(0.5, 3.0, n_steps)  
        gapSize_series = np.random.normal(0, 1, n_steps)  
  
        # 隱藏的市場規則  
        hidden_signal = np.std(sdWidth_series) * 2.0 + np.max(gapSize_series) * 1.5  
        hidden_signal += np.random.normal(0, 0.5)  
        hidden_signals.append(hidden_signal)  # 先存起來  
  
        for step in range(n_steps):  
            data_list.append({  
                'run_id': run_id,  
                'time_step': step,  
                'sdWidth': sdWidth_series[step],  
                'gapSize': gapSize_series[step]  
            })  
  
    # 第二階段：統一計算全市場的 80% 門檻，決定誰是 1 誰是 0    threshold = np.percentile(hidden_signals, 80)  
    y_dict = {i: (1 if val > threshold else 0) for i, val in enumerate(hidden_signals)}  
  
    df_ts = pd.DataFrame(data_list)  
    y_series = pd.Series(y_dict)  
  
    print(f"📊 標籤分佈檢查: 0(盤整/賠) = {(y_series == 0).sum()} 筆, 1(暴漲) = {(y_series == 1).sum()} 筆")  
  
    return df_ts, y_series  
  
  
# ==========================================  
# 核心修正：將主要邏輯包裝在 main 函式中  
# ==========================================  
def main():  
    # 1. 取得資料  
    df_ts, y_target = generate_ts_data()  
  
    # ==========================================  
    # 2. 煉金術士登場：tsfresh 自動特徵發明  
    # ==========================================  
    print("\n🧙‍♂️ tsfresh 煉金術士開始施法，暴力展開時間序列特徵...")  
  
    # 確保在 Windows 下執行 multiprocessing 不會崩潰  
    extracted_features = extract_features(  
        df_ts,  
        column_id="run_id",  
        column_sort="time_step",  
        disable_progressbar=False,  
        n_jobs=4  # 您可以根據電腦核心數調整，如果還是報錯，可以設為 1 (關閉多核心)  
    )  
  
    impute(extracted_features)  
    print(f"✨ 煉金完成！擴增成了 {extracted_features.shape[1]} 個複雜特徵！")  
  
    print("🧹 tsfresh 進行初步雜訊過濾...")  
    X_filtered = select_features(extracted_features, y_target)  
    print(f"✅ 保留了 {X_filtered.shape[1]} 個特徵。")  
  
    # ==========================================  
    # 3. 試金石登場：Optuna + XGBoost 尋找最佳配方  
    # ==========================================  
    X_train, X_test, y_train, y_test = train_test_split(X_filtered, y_target, test_size=0.2, random_state=42)  
  
    def objective(trial):  
        param = {  
            "n_estimators": trial.suggest_int("n_estimators", 50, 150),  
            "max_depth": trial.suggest_int("max_depth", 3, 6),  
            "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.1, log=True),  
            "scale_pos_weight": trial.suggest_float("scale_pos_weight", 1.0, 5.0),  
            "reg_alpha": trial.suggest_float("reg_alpha", 0.1, 10.0, log=True),  
            "random_state": 42,  
            "n_jobs": -1  
        }  
  
        model = xgb.XGBClassifier(**param)  
        model.fit(X_train, y_train)  
  
        entry_threshold = trial.suggest_float("entry_threshold", 0.5, 0.8)  
        preds_prob = model.predict_proba(X_test)[:, 1]  
        preds_class = (preds_prob > entry_threshold).astype(int)  
  
        return precision_score(y_test, preds_class, zero_division=0)  
  
    print("\n🚀 啟動 Optuna 尋寶...")  
    study = optuna.create_study(direction="maximize")  
    study.optimize(objective, n_trials=30)  
  
    print("\n" + "=" * 50)  
    print(f"🥇 最佳測試集 Precision: {study.best_value:.4f}")  
    print("=" * 50)  
  
    # ==========================================  
    # 4. 驗證時刻  
    # ==========================================  
    print("\n🔍 揭曉最強的煉金配方...")  
    best_params = study.best_params  
    best_threshold = best_params.pop("entry_threshold")  
    best_params["random_state"] = 42  
  
    final_model = xgb.XGBClassifier(**best_params)  
    final_model.fit(X_train, y_train)  
  
    importances = final_model.feature_importances_  
    feature_names = X_filtered.columns  
  
    feature_importance_df = pd.DataFrame({  
        'Feature': feature_names,  
        'Importance': importances  
    }).sort_values(by='Importance', ascending=False)  
  
    survivors = feature_importance_df[feature_importance_df['Importance'] > 0]  
  
    print(f"\n🔥 從 {extracted_features.shape[1]} 個特徵中，最終存活了 {len(survivors)} 個黃金特徵！")  
    print("\n👑 倖存特徵 Top 10 排行榜：")  
    print(survivors.head(10).to_string(index=False))  
  
  
# ==========================================  
# Windows 系統必須加這行防護罩！  
# ==========================================  
if __name__ == '__main__':  
    # 加上這行，避免 Windows 多進程報錯  
    import multiprocessing  
  
    multiprocessing.freeze_support()  
  
    # 啟動主程式  
    main()
```

這是量化特徵工程的終極型態。我們不再自己發明數學公式，而是把整塊時間序列資料（例如連續 16 個時間步長的 `sdWidth` 和 `gapSize`）交給機器，讓它自己「無中生有」出複雜的指標。

### 核心運作流程

1. **暴力展開 (`tsfresh.extract_features`)**： 將原本只有幾個基礎屬性的時間序列，瞬間擴增成上千個包含傅立葉轉換 (FFT)、連續小波轉換 (CWT)、偏度 (Skewness)、線性趨勢標準誤等複雜數學公式的特徵。
    
2. **統計初篩 (`tsfresh.select_features`)**： 利用 p-value 假設檢定，先剃除掉對目標標籤（PnL）絕對沒有統計顯著性的特徵，完成初步瘦身。
    
3. **Optuna 試金石**： 將初篩後的特徵送入 Optuna，再次利用 `reg_alpha` 進行最終的無情淘汰與參數收斂。
    

### ⚠️ Windows 實戰避坑指南

在 Windows 系統中執行 `tsfresh` 的多核心運算時，極易觸發 `RuntimeError` 無窮迴圈崩潰。 **解法**：必須將主要執行邏輯包裝在 `main()` 函式中，並在程式進入點宣告 `multiprocessing.freeze_support()`，才能確保多進程安全啟動。

### 神級配方的解讀

最終排行榜上出現的特徵，就是機器為市場量身打造的專屬指標：

- `gapSize__maximum`：該區間內的最大缺口。
    
- `gapSize__fft_coefficient__attr_"abs"__coeff_2`：缺口變化的傅立葉轉換低頻震盪週期。