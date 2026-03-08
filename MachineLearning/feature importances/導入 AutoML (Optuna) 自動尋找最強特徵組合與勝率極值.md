



# 第二篇：【量化交易實戰】告別手動煉丹！導入 AutoML (Optuna) 自動尋找最強特徵組合與勝率極值

### 前言：知道了重要性，然後呢？

在上一篇我們學會了剔除無用特徵。但實戰中最大的痛點來了：**我們該如何組合剩下的黃金特徵？**

「成交量要乘上 0.8 還是 1.2？」「這兩個指標要不要套用 `sin()` 函數？」如果您還在手動修改 Java 程式碼、重新編譯、再跑 Python 看勝率，這種「手動煉丹」的效率絕對跟不上千變萬化的市場。

這時，我們需要引入 **AutoML (自動化機器學習)** 與最強的尋寶引擎：**Optuna**。

### AutoML 是如何運作的？

AutoML 不是魔法，它是一個嚴格的試錯教練。透過 **貝氏最佳化 (Bayesian Optimization)**，它會根據前幾次實驗的回饋，聰明地推測出下一組最有可能提高勝率的「參數組合」。

它最後會給你兩樣東西：

1. **一個訓練好的黑盒子模型** (如 XGBoost)。
    
2. **一張人類完全看得懂的「最佳配方表」** (Best Parameters)。
    

### Optuna 實戰：自動尋找最佳特徵公式

在這個範例中，我們不手動寫死公式，而是把「要不要加 sin」和「權重該放多少」設定為變數，讓 Optuna 自己去跑幾十次迴圈找答案：



```Python
import optuna
import xgboost as xgb
import numpy as np
from sklearn.metrics import precision_score
from sklearn.model_selection import train_test_split

def objective(trial):
    # 1. 讓 Optuna 自動決定特徵的組合方式 (Automated Feature Engineering)
    use_sin_transform = trial.suggest_categorical("use_sin_transform", [True, False])
    vol_weight = trial.suggest_float("vol_weight", 0.1, 5.0)
    
    # 動態生成新特徵
    X = df[['volume_rate', 'flat_time_rate']].copy()
    if use_sin_transform:
        X['magic_feature'] = (X['volume_rate'] * vol_weight) * np.sin(X['flat_time_rate'])
    else:
        X['magic_feature'] = (X['volume_rate'] * vol_weight) * X['flat_time_rate']
        
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, shuffle=False)

    # 2. 讓 Optuna 尋找最佳的模型參數
    param = {
        "max_depth": trial.suggest_int("max_depth", 3, 7),
        "learning_rate": trial.suggest_float("learning_rate", 0.01, 0.2, log=True),
        "scale_pos_weight": trial.suggest_float("scale_pos_weight", 1.0, 10.0) # 自動處理不平衡資料！
    }

    # 3. 訓練並回傳我們最在乎的指標：Precision (做多勝率)
    model = xgb.XGBClassifier(**param)
    model.fit(X_train, y_train)
    
    preds = model.predict(X_test)
    return precision_score(y_test, preds, zero_division=0)

# 啟動尋寶引擎，目標是「最大化」勝率
study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=50)

print("🏆 最佳配方出爐：", study.best_params)
```

### 拿到 AutoML 的配方後，該怎麼做？

當 Optuna 跑完後，終端機會印出類似這樣的結果：

`{'use_sin_transform': True, 'vol_weight': 2.34, 'max_depth': 5}`

這就是量化工程師的聖杯！這代表你不必依賴 Python 那個龐大且難以解釋的 `.pkl` 模型檔。你可以直接轉身回到你的核心 Java 交易系統中，把這個配方寫進邏輯裡：**「只要成交量變化率乘上 2.34 再乘上持平時間的 sin 值大於某個門檻，就開多單！」**

AutoML 幫我們省下了數百小時的試錯時間，讓我們能專注於開發更穩定、更低延遲的交易系統。

---

請問您覺得這兩篇的深淺度如何？需要我為您進一步擴寫哪一個段落（例如 Optuna 如何整合 MLflow 追蹤）嗎？