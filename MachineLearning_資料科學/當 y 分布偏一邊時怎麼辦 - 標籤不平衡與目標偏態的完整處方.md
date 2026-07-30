

# 【量化交易實戰】當 y 分布偏一邊時怎麼辦？標籤不平衡與目標偏態的完整處方

### 前言：你的模型可能只是學會了「永遠猜 0」

打開標籤分佈一看：

```
📊 標籤分佈: 不進場(0) = 9,412 筆, 獲利進場(1) = 588 筆
```

這種 y 擠在一個區間的情況，在量化交易幾乎是常態——**能賺錢的訊號本來就稀有**。此時直接開訓，模型會學到一個作弊解：**全部猜 0，準確率立刻 94%**。你看著漂亮的 accuracy 上線，實盤卻一次單都不進。

「y 偏一邊」其實是兩個不同的病，處方完全不同：

| 病症 | 長相 | 章節 |
|---|---|---|
| **分類標籤不平衡** | y 是 0/1，其中一類佔壓倒性多數 | 第一部 |
| **迴歸目標偏態** | y 是連續值，擠在小區間 + 拖長尾（如報酬率） | 第二部 |

---

## 第一部：分類 —— 標籤不平衡 (Class Imbalance)

### Step 0：先換掉會騙人的量尺

不平衡資料下 **accuracy 完全失效**。先把評估指標換成看得見稀有類的：

```Python
from sklearn.metrics import (
    classification_report, confusion_matrix, average_precision_score
)

y_pred  = model.predict(X_test)
y_proba = model.predict_proba(X_test)[:, 1]

print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred, digits=3))
print(f"PR-AUC: {average_precision_score(y_test, y_proba):.3f}")
```

- **Precision（精確率）**：模型喊進場的訊號裡，幾成真的賺？→ 決定你的**勝率**
- **Recall（召回率）**：所有賺錢機會裡，模型抓到幾成？→ 決定你的**交易頻率**
- **PR-AUC**：不平衡資料下比 ROC-AUC 誠實得多——ROC-AUC 會被大量的 0 灌水，看起來 0.9 其實稀有類抓得一塌糊塗。

> 量化場景通常 **Precision 優先於 Recall**：錯過機會只是少賺，錯誤進場是真賠錢。

### 處方一：類別權重（首選，零副作用）

不動資料，只告訴演算法「答錯稀有類，罰十倍」：

```Python
from sklearn.ensemble import RandomForestClassifier

# Random Forest：自動按類別比例反向加權
model = RandomForestClassifier(
    n_estimators=300,
    class_weight='balanced',   # 或 'balanced_subsample'（每棵樹各自平衡，通常更穩）
    random_state=42,
)
```

```Python
import xgboost as xgb

# XGBoost：scale_pos_weight = 負樣本數 / 正樣本數
ratio = (y_train == 0).sum() / (y_train == 1).sum()
model = xgb.XGBClassifier(scale_pos_weight=ratio)
```

`scale_pos_weight` 也可以交給 Optuna 自動搜（見 [[AutoML-01  導入 AutoML (Optuna) 自動尋找最強特徵組合與勝率極值]]），因為「數學上的平衡比例」不一定是「賺賠比上的最佳比例」。

### 處方二：門檻搬移（Threshold Moving）

`predict()` 預設用 0.5 切分——這個數字沒有任何神聖性。不平衡資料下，直接對 `predict_proba` 自訂門檻往往比重採樣更有效：

```Python
import numpy as np
from sklearn.metrics import precision_recall_curve

prec, rec, thresholds = precision_recall_curve(y_test, y_proba)

# 例：要求勝率至少 60%，在此前提下找 recall 最高的門檻
mask = prec[:-1] >= 0.60
best_t = thresholds[mask][np.argmax(rec[:-1][mask])]
print(f"進場門檻: {best_t:.3f}")

y_pred_custom = (y_proba >= best_t).astype(int)
```

這招在交易上特別自然：**門檻拉高 = 只打最有把握的球**，訊號變少但更準。

### 處方三：重採樣（Resampling）—— 小心使用

```Python
from imblearn.over_sampling import SMOTE
from imblearn.under_sampling import RandomUnderSampler

# 過採樣：SMOTE 在少數類樣本之間內插出合成樣本
X_res, y_res = SMOTE(random_state=42).fit_resample(X_train, y_train)

# 欠採樣：隨機丟棄多數類（資料量大時的快速選項）
X_res, y_res = RandomUnderSampler(random_state=42).fit_resample(X_train, y_train)
```

**三條鐵律：**

1. **只對訓練集做**。測試集必須保持真實世界的原始分布，否則你評估的是一個不存在的市場。
2. **時間序列慎用 SMOTE**：內插出來的合成樣本混合了不同時間點的資訊，相鄰樣本又高度自相關，等於變相洩漏。時序資料優先用處方一、二；真要用，只在按時間切好的訓練段內做。
3. **重採樣後 `predict_proba` 已失真**：模型眼中的世界是 50/50，輸出的機率不再是真實頻率。要拿機率當下注大小（如 Kelly），需先做校準（`CalibratedClassifierCV`）。

### 處方四：回頭改標籤設計（治本）

如果 1 的比例低到千分之幾，先別急著折磨演算法——**問題可能出在標籤定義**：

- 獲利門檻是不是設太苛？（+5% 才算 1 → 改 +2%）
- 持有窗口是不是太短？（10 根 K 棒 → 放寬到 30 根）
- 能不能改成三分類（漲/盤/跌）讓資訊更平均？

標籤設計挪一格，分布常常就從 99:1 變成 85:15，後面所有處方的壓力都小一半。

---

## 第二部：迴歸 —— 目標偏態 (Skewed Target)

y 是連續值（如未來報酬率、波動率）且擠在一個小區間、拖著長尾時，MSE 類損失會被尾巴上的極端值綁架，模型對「大多數樣本所在的區間」反而學得很差。

### 處方一：對 y 做壓縮轉換，預測完再反轉

```Python
import numpy as np
from sklearn.compose import TransformedTargetRegressor
from sklearn.ensemble import RandomForestRegressor

# log1p 適用 y >= 0；含負值（如報酬率）改用 Yeo-Johnson
model = TransformedTargetRegressor(
    regressor=RandomForestRegressor(n_estimators=300, random_state=42),
    func=np.log1p,
    inverse_func=np.expm1,   # 預測值自動反轉回原始尺度
)
model.fit(X_train, y_train)
```

含負值的 y 用 Yeo-Johnson（Box-Cox 只吃正值）：

```Python
from sklearn.preprocessing import PowerTransformer

model = TransformedTargetRegressor(
    regressor=RandomForestRegressor(n_estimators=300, random_state=42),
    transformer=PowerTransformer(method='yeo-johnson'),
)
```

> 注意：log 轉換後用 MSE 訓練，反轉回來的預測是**中位數傾向**而非平均數傾向，會系統性低估大值。對交易來說通常反而是優點（保守），但要心裡有數。

### 處方二：換損失函數，不動 y

- **Huber loss**：小誤差用 MSE、大誤差用 MAE，天生抗長尾極端值。
  ```Python
  from sklearn.ensemble import HistGradientBoostingRegressor
  model = HistGradientBoostingRegressor(loss='absolute_error')  # 或 XGBoost objective='reg:pseudohubererror'
  ```
- **Quantile loss**：直接預測分位數（例如第 20 百分位的保守報酬估計），比預測平均值更貼近風控需求。

### 處方三：放棄迴歸，改成分類

量化實戰中常見的降維打擊：你其實不需要知道「會漲 3.7%」，只需要知道「會不會漲超過手續費+滑價」。把連續 y 按門檻或分位數切桶，轉成分類問題，回到第一部的世界——樹模型處理分類的不平衡工具鏈成熟得多。

```Python
y_cls = (y > 0.002).astype(int)   # 漲幅超過成本才算 1
```

---

## 快速決策表

| 你的情況 | 首選處方 |
|---|---|
| 分類、不平衡在 10:1 以內 | `class_weight` / `scale_pos_weight` + 換 PR-AUC 評估 |
| 分類、要控制勝率 | 門檻搬移（`precision_recall_curve` 找門檻） |
| 分類、極端不平衡（>100:1） | 先回頭改標籤設計，再上權重 |
| 迴歸、y 非負長尾 | `log1p` + `TransformedTargetRegressor` |
| 迴歸、y 含負值 | Yeo-Johnson 或 Huber loss |
| 迴歸、但決策其實是二元的 | 切桶轉分類 |

## 結論

- y 偏一邊不是罕見病,是量化的日常——**先換指標（PR-AUC / Precision / Recall），再談處方**，否則你連病情都看不見。
- 處方優先序：**改標籤設計 > 類別權重/門檻搬移 > 重採樣**。越不動資料的方法副作用越小。
- 重採樣三鐵律：只動訓練集、時序慎用 SMOTE、動過的機率要先校準才能當下注依據。
- 特徵 X 的偏態處理是另一個主題，見 [[EDA 解密 Histogram Grid —— 揪出摧毀模型的「極端值地雷」]]。
