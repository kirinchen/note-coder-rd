

# 機器學習雙雄對決：隨機森林 (Random Forest) vs. XGBoost 實戰教學

在資料科學的江湖裡，有兩個門派長期佔據著「最強演算法」的寶座。一個是主打「團結力量大」的 **隨機森林 (Random Forest)**，另一個是主打「知錯能改」的 **XGBoost**。

今天這篇文章不講艱澀的數學公式，我們直接用一個最生活化的案例——**「預測房價」**，帶你手把手寫出這兩個模型，並看看誰比較準！

## 1. 核心概念：它們是怎麼思考的？

想像你要估算一間房子的價格，這兩個模型會怎麼做？

### 🌲 隨機森林 (Random Forest)：三個臭皮匠，勝過一個諸葛亮

它的邏輯是 **「Bagging (袋裝法)」**。

想像你找了 100 位房仲來估價：

- **房仲 A** 只看坪數。
    
- **房仲 B** 只看屋齡。
    
- **房仲 C** 只看離捷運站多遠。
    
    每個房仲都有一點片面，但最後我們把這 100 個人的估價**「取平均值」**。這樣做的好處是，就算某個房仲看走眼，也會被其他人平衡掉，所以結果非常**穩定**，不易出錯。
    

### 🚀 XGBoost：知錯能改，善莫大焉

它的邏輯是 **「Boosting (提升法)」**。

想像只有一位超級認真的學生在練習估價：

- **第一輪：** 他亂猜一通，發現「大坪數的房子」估太低了。
    
- **第二輪：** 他**專注於修正**上一輪的錯誤，這次特別注意坪數，結果發現「老房子」估太高了。
    
- **第三輪：** 他再次修正，針對老房子降價...
    
    它是一步一步**「補強弱點」**，直到錯誤率降到最低。這就是為什麼 XGBoost 通常比較**準**的原因。
    

---

## 2. 實戰演練：房價預測器

為了方便大家練習，我們不下載複雜的真實數據，直接用 Python 生成一份模擬的房產資料。

### 第一步：準備環境與製造數據

我們假設房價由三個因素決定：**坪數 (Size)**、**房間數 (Rooms)**、**屋齡 (Age)**。



```Python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error

# 1. 製造假資料 (Mock Data)
# 設定隨機種子，保證每次跑結果一樣
np.random.seed(42)

n_samples = 1000
data = {
    'Size_sqft': np.random.randint(500, 3500, n_samples),   # 坪數 (平方英尺)
    'Num_Rooms': np.random.randint(1, 6, n_samples),        # 房間數
    'Age_Years': np.random.randint(0, 50, n_samples),       # 屋齡
}

df = pd.DataFrame(data)

# 2. 設定真實的房價公式 (這是上帝視角，模型不知道這個公式)
# 房價 = 坪數*150 + 房間*10000 - 屋齡*500 + 一點點隨機波動
df['Price'] = (
    df['Size_sqft'] * 150 + 
    df['Num_Rooms'] * 10000 - 
    df['Age_Years'] * 500 + 
    np.random.randint(-5000, 5000, n_samples) # 雜訊
)

# 3. 切分訓練集與測試集 (80% 訓練, 20% 考試)
X = df.drop('Price', axis=1) # 題目
y = df['Price']              # 答案

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

print(f"資料準備完成！我們有 {len(X_train)} 筆資料用來訓練。")
print(X_train.head())
```

---

### 第二步：派出選手 A —— 隨機森林

隨機森林是 `scikit-learn` 套件內建的，非常容易上手，幾乎不需要調整參數就能跑得不錯。



```Python
from sklearn.ensemble import RandomForestRegressor

# 1. 建立模型 (叫出 100 個房仲)
rf_model = RandomForestRegressor(n_estimators=100, random_state=42)

# 2. 訓練模型 (讓房仲看歷史資料)
rf_model.fit(X_train, y_train)

# 3. 預測 (考試)
rf_preds = rf_model.predict(X_test)

# 4. 驗收成績 (平均絕對誤差 MAE)
rf_mae = mean_absolute_error(y_test, rf_preds)
print(f"🌲 隨機森林的平均誤差: ${rf_mae:,.0f}")
```

---

### 第三步：派出選手 B —— XGBoost

XGBoost 需要額外安裝 (`pip install xgboost`)，它的威力在於對數據特徵的細膩捕捉。



```Python
import xgboost as xgb

# 1. 建立模型
xgb_model = xgb.XGBRegressor(
    n_estimators=100,     # 訓練 100 輪
    learning_rate=0.1,    # 學習率 (每一步修正多少)
    max_depth=3,          # 樹的深度 (不要太深，避免死背)
    random_state=42
)

# 2. 訓練模型
xgb_model.fit(X_train, y_train)

# 3. 預測
xgb_preds = xgb_model.predict(X_test)

# 4. 驗收成績
xgb_mae = mean_absolute_error(y_test, xgb_preds)
print(f"🚀 XGBoost 的平均誤差:  ${xgb_mae:,.0f}")
```

---

## 3. 賽後分析：誰贏了？為什麼？

執行上述程式碼後，你可能會看到類似這樣的結果（數值可能會有些微差異）：

- **隨機森林誤差：** $3,200
    
- **XGBoost 誤差：** $2,800
    

在這種結構化表格數據（Excel 類型的資料）中，**XGBoost 通常會略勝一籌**。因為它會針對前一棵樹預測不好的「困難案例」（例如：超大坪數但很舊的房子）進行加強訓練。

### 特徵重要性 (Feature Importance)

最後，我們可以問問模型：「你覺得決定房價最重要的因素是什麼？」這在商業上非常有價值。



```Python
import matplotlib.pyplot as plt

# 畫出 XGBoost 認為的特徵重要性
xgb.plot_importance(xgb_model)
plt.title("XGBoost 認為的關鍵因素")
plt.show()
```

你會發現模型很聰明，它會自動發現 `Size_sqft`（坪數）的分數最高，這完全符合我們一開始設定的邏輯（坪數乘的係數最大）。

---

## 結論：我該用哪一個？

- **新手入門 / 快速驗證：** 選 **隨機森林 (Random Forest)**。它不需要調什麼參數，丟進去通常就有 80 分的水準，而且不容易過擬合。
    
- **追求準確度 / 比賽 / 商業落地：** 選 **XGBoost**。雖然它參數比較多（學習率、樹深...），但調校得當的話，它幾乎是目前最強的表格數據演算法。
    

希望這篇文章能讓你理解這兩個 AI 神器的區別！下次遇到數據預測問題，不妨兩個都試試看！
