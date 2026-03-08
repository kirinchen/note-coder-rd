

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

### 結論與避坑指南

看到某個特徵分數很高，**不代表它能幫你賺錢**，只代表「模型在做決定時很依賴它」。

Feature Importance 最大的實戰用途是 **「減肥與降噪」**。透過這張圖表，您可以果斷把排名墊底的垃圾特徵（例如對勝率毫無幫助的星期幾、或過度落後的均線）從系統中剔除，讓未來的模型跑得更快、更精準！
