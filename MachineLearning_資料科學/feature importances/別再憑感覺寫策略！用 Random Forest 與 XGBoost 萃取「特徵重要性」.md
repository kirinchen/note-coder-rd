

# 第一篇：【量化交易實戰】別再憑感覺寫策略！用 Random Forest 與 XGBoost 萃取「特徵重要性」

### 前言：我們真的知道哪個指標有用嗎？

在開發量化交易策略時，我們常會塞入各種技術指標：RSI、MACD、成交量變化率，甚至自創的複雜公式。但當模型預測失敗時，我們往往不知道是哪個環節出了問題。

與其憑直覺增減條件，不如讓機器親自告訴我們：「在過去的歷史回測中，它到底看重哪些數據？」這就是 **特徵重要性 (Feature Importance)** 的核心價值。

### 什麼是 Feature Importance？

簡單來說，特徵重要性就是模型的「讀書時間分配表」。當我們訓練一棵決策樹（如 Random Forest 或 XGBoost）時，演算法會不斷尋找能把「賺錢」與「賠錢」分得最乾淨的特徵。被用來切分最多次、降低最多亂度（Impurity）的特徵，它的重要性分數就會越高。

### Python 實戰程式碼

以下是一個極簡的實作範例，使用 Random Forest 來訓練資料，並透過 Seaborn 畫出重要性排行：



```Python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.ensemble import RandomForestClassifier

# 1. 準備資料 (X 為特徵，y 為進場是否獲利的標籤 1或0)
# df = pd.read_parquet('your_trading_data.parquet')
X = df.drop(columns=['target_label', 'time'])
y = df['target_label']

# 2. 訓練隨機森林模型
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X, y)

# 3. 提取特徵重要性
importance = model.feature_importances_
feature_names = X.columns

# 4. 視覺化排行 (使用 Seaborn)
plt.figure(figsize=(10, 6))
sns.barplot(
    x=importance, 
    y=feature_names, 
    hue=feature_names, 
    palette='viridis', 
    legend=False
)
plt.title('量化策略特徵重要性排行')
plt.xlabel('重要性分數 (越高代表模型越依賴該特徵)')
plt.show()
```

### 內建重要性的先天缺陷

上面用的 `model.feature_importances_` 是 **Impurity-based Importance**（基於不純度下降），它是「訓練時的副產品」，快但有兩個先天問題：

1. **偏愛高基數特徵**：連續值、取值種類多的特徵（如成交量、價格）天生有更多切分點可選，分數會被高估；反之像「星期幾」這種低基數特徵容易被低估——分數低不一定是真的沒用。
2. **只反映訓練集**：它衡量的是模型「訓練時切了多少次」，就算某特徵只是幫模型背答案（過擬合噪音），分數一樣會很高。**它完全沒有回答「這個特徵對泛化預測有沒有幫助」。**

### Permutation Importance：更誠實的量法

**Permutation Importance（排列重要性）** 的邏輯很直觀：

> 把某一欄特徵的值**隨機打亂**（切斷它與標籤的關係），看模型在**驗證集**上的分數掉多少。掉越多，代表模型越依賴這個特徵做出正確預測。

因為是在「模型訓練完之後、用沒看過的資料」衡量，它天然避開了上面兩個缺陷：

```Python
from sklearn.inspection import permutation_importance
from sklearn.model_selection import train_test_split

# 時間序列資料切分請勿 shuffle，避免用未來資料訓練
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, shuffle=False
)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 在「測試集」上打亂每個特徵 10 次，觀察分數下降量
perm = permutation_importance(
    model, X_test, y_test,
    n_repeats=10, random_state=42, scoring='accuracy'
)

perm_df = pd.DataFrame({
    'feature': X.columns,
    'importance_mean': perm.importances_mean,
    'importance_std': perm.importances_std,
}).sort_values('importance_mean', ascending=False)

print(perm_df)
```

**判讀重點：**

- `importance_mean` ≈ 0 甚至為負 → 這個特徵對泛化預測**毫無貢獻**，可以剔除。
- `importance_std` 很大（跟 mean 同數量級）→ 分數不穩定，別急著下結論，增加 `n_repeats` 或換幾個 random seed 再看。
- 若某特徵在 **impurity 排行很高、permutation 排行卻墊底** → 高機率是模型拿它在背訓練集答案，這正是過擬合的訊號。

### 兩種量法對照表

| | Impurity-based | Permutation |
|---|---|---|
| 計算成本 | 免費（訓練副產品） | 貴（每特徵重算 n_repeats 次） |
| 衡量對象 | 訓練集切分貢獻 | 驗證/測試集泛化貢獻 |
| 高基數偏差 | 有 | 無 |
| 能否用於任何模型 | 僅樹模型 | 任何模型（model-agnostic） |
| 共線性下的表現 | 分數被瓜分 | 分數被低估（見下方陷阱） |

### 解讀陷阱大全

1. **重要 ≠ 能賺錢**：分數只代表「模型依賴它」。若標籤設計有問題（例如進出場定義偏差），模型依賴的是垃圾訊號。
2. **共線性會騙人**：兩個高度相關的特徵（如 5MA 與 10MA），impurity 會把功勞**隨機瓜分**給其中一個；permutation 則會**兩個都低估**——因為打亂其中一個時，模型還能靠另一個補位，分數幾乎不掉。你可能因此誤刪一對其實很有用的特徵。
   - **解法**：先用 Correlation Heatmap（見 [[EDA 特徵熱力圖 (Correlation Heatmap)]]）做特徵聚類，每群只留一個代表再跑重要性。
3. **資料洩漏的特徵永遠是第一名**：如果某特徵分數高到不合理（一枝獨秀），先懷疑它偷看了未來資料（如用到當根 K 棒收盤後才知道的值），而不是慶祝找到聖杯。
4. **分數不穩定**：換一個 `random_state` 排行就大洗牌，代表特徵之間差異其實在噪音範圍內，排行本身沒有意義。務必用多 seed / 多 fold 驗證排行穩定性。
5. **在訓練集上跑 permutation 沒有意義**：一定要用模型沒看過的資料，否則和 impurity-based 一樣量到的是記憶力而不是預測力。
6. **時間序列別亂 shuffle**：切分訓練/測試集要按時間切（`shuffle=False` 或 walk-forward），否則重要性本身就是建立在洩漏之上。

### 結論與實戰流程

看到某個特徵分數很高，**不代表它能幫你賺錢**，只代表「模型在做決定時很依賴它」。

Feature Importance 最大的實戰用途是 **「減肥與降噪」**。建議流程：

1. 先用 Heatmap 對特徵做共線性分群，每群留一個代表。
2. 用 impurity-based 快速掃一輪，砍掉明顯墊底的垃圾特徵。
3. 對存活者跑 permutation importance（測試集、多 seed），確認泛化貢獻。
4. 對「impurity 高、permutation 低」的特徵保持警覺——那是過擬合的指紋。

把排名墊底的垃圾特徵從系統中剔除，讓未來的模型跑得更快、更精準！

> 下一步：單一特徵排完強弱後，試著讓特徵彼此組合（a×b、a÷b、a×cos(b)…）再量一次 → [[特徵交互強弱分析 - 從 a×b 到符號迴歸,挖掘單一指標看不見的訊號]]
