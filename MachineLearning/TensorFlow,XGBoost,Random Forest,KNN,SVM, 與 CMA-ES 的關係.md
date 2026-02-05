

# 迷航指南：TensorFlow,XGBoost,Random Forest,KNN,SVM, 與 CMA-ES 的關係

### —— 一次搞懂機器學習、深度學習與最佳化演算法的地圖

在踏入量化交易或 AI 開發的領域時，我們常會被各種名詞轟炸：TensorFlow, Keras, Scikit-learn, XGBoost, CMA-ES... 它們有些是「工具」，有些是「演算法」，有些是「數學概念」。

你是否也有這樣的疑問：

> _"TensorFlow 是不是只能拿來跑 LSTM？隨機森林跟 XGBoost 是另一掛的嗎？CMA-ES 又是什麼，它算機器學習嗎？"_

這篇文章將為您拆解這張 AI 演圖。

---

## 1. 深度學習 vs. 傳統機器學習：工具的楚河漢界

首先，回答您的第一個核心問題：**TensorFlow 是不是只能用在 LSTM 或 MLP？**

### **TensorFlow / PyTorch**：深度學習的重型武器

- **主要戰場**：類神經網路 (Neural Networks)。包括 **MLP** (多層感知機), **CNN** (卷積神經網路), **LSTM/Transformer** (序列模型)。
    
- **為什麼用它**：這些模型需要大量的矩陣運算 (Matrix Multiplication) 和自動微分 (Auto-differentiation) 來進行反向傳播 (Backpropagation)。TensorFlow 就是為了這件事而生的。
    
- **例外**：雖然 TensorFlow 也有 `TF Decision Forests` 可以跑隨機森林，但在業界慣例中，我們 **99% 的情況下** 只會用它來跑深度學習模型。
    

### **Scikit-learn / XGBoost**：傳統機器學習的瑞士軍刀

隨機森林 (Random Forest)、KNN (K-Nearest Neighbors)、SVM 等演算法，通常被稱為「傳統機器學習 (Classical Machine Learning)」。

- **隨機森林 (Random Forest)** & **KNN**：通常使用 Python 的 **`scikit-learn`** 套件。
    
- **XGBoost / LightGBM / CatBoost**：這些是「梯度提升樹 (Gradient Boosting Decision Tree)」的變體。因為太強大且需要極致效能，它們通常由 **獨立的套件** 提供 (例如 `import xgboost`)，不隸屬於 TensorFlow，也不完全隸屬於 scikit-learn (雖然介面相容)。
    

**小結：**

- 要跑 **LSTM / MLP** ➔ 用 **TensorFlow** 或 **PyTorch**。
    
- 要跑 **Random Forest / KNN** ➔ 用 **Scikit-learn**。
    
- 要跑 **Gradient Boosting** ➔ 用 **XGBoost** 獨立套件。
    

---

## 2. 它們全部都叫「機器學習」嗎？

是的，廣義來說，它們都在 **人工智慧 (AI)** ➔ **機器學習 (Machine Learning)** 的大傘下。

但我們通常會這樣細分：

|**分類**|**代表演算法**|**特性**|**適用場景 (交易為例)**|
|---|---|---|---|
|**深度學習 (Deep Learning)**|LSTM, MLP, Transformer|黑盒子、參數極多、能自動提取特徵。|預測股價走勢、分析K線圖形、NLP 分析財報新聞。|
|**傳統監督式學習 (Supervised ML)**|XGBoost, Random Forest, KNN|可解釋性較高、處理表格資料 (Tabular Data) 強項。|根據 MACD, RSI 等指標預測漲跌機率。|
|**非監督式學習 (Unsupervised ML)**|Pattern Matching (特徵空間法), K-Means|沒有標準答案，尋找資料結構。|尋找歷史上相似的 K 線型態 (Pattern Matching)。|

---

## 3. 異類突起：CMA-ES 也是機器學習嗎？

**CMA-ES (Covariance Matrix Adaptation Evolution Strategy)** 的定位比較特殊。

嚴格來說，它屬於 **演化計算 (Evolutionary Computation)** 或 **黑盒最佳化 (Black-box Optimization)**，而不是典型的「監督式機器學習」。

### 關鍵差異：

- **機器學習 (ML)** 的目標是 **「預測 (Prediction)」**。
    
    - _Input:_ 歷史數據 $\rightarrow$ _Model_ $\rightarrow$ _Output:_ 明天會漲還是跌？
        
- **CMA-ES** 的目標是 **「搜尋 (Search)」**。
    
    - _Input:_ 一個評分標準 (例如：獲利金額) $\rightarrow$ _CMA-ES_ $\rightarrow$ _Output:_ 給出能讓獲利最大化的**參數組合**。
        

**所以，CMA-ES 算是 ML 嗎？**

在廣義的 AI 領域中，它是「最佳化演算法」。但在現代用法中，它常被視為 **強化學習 (Reinforcement Learning)** 的一種替代方案，或者被歸類為 AI 中的 Search Algorithm。

---

## 4. 終極圖解：它們如何在其交易系統中協作？

這才是最精彩的部分。在一個成熟的 **量化交易系統 (Quantitative Trading System)** 中，這些技術往往是**串聯**使用的，而不是互斥的。

想像您的交易機器人是一個團隊：

### 角色 A：分析師 (Deep Learning / Traditional ML)

- **成員**：TensorFlow (LSTM) 或 XGBoost。
    
- **任務**：看著過去的價格，預測未來的走勢。
    
- **產出**：「根據現在的盤勢，我有 70% 信心認為下一小時會漲。」
    

### 角色 B：圖形比對員 (Pattern Matching)

- **成員**：KNN 或特徵空間法。
    
- **任務**：翻閱歷史舊帳。
    
- **產出**：「現在這個 M 頭型態，跟 2021 年 5 月崩盤前的樣子有 95% 像！」
    

### 角色 C：教練 / 調參師 (CMA-ES)

- **成員**：CMA-ES 演算法。
    
- **任務**：它不看盤，它看的是「策略的績效」。它負責調整角色 A 和 角色 B 的參數。
    
- **運作方式**：
    
    - CMA-ES 會問：「如果我把 RSI 的週期從 14 改成 20，把停損點從 5% 改成 3%，賺的錢會變多嗎？」
        
    - 它會生成數百種參數組合，丟給回測系統跑，然後根據結果「進化」出最強的參數。
        

---

## 總結

1. **TensorFlow** 是深度學習的專武 (LSTM, MLP)，別拿它殺雞焉用牛刀去跑隨機森林。
    
2. **Random Forest, KNN, XGBoost** 是傳統機器學習的強者，通常使用 Scikit-learn 或 XGBoost 獨立套件。
    
3. **CMA-ES** 不是用來預測行情的，它是用來**尋找最強參數**的最佳化演算法。
    

在您的架構中，您可能會用 **MLflow** 來管理這一切：用 MLflow 記錄 XGBoost 的訓練結果，同時也用 MLflow 記錄 CMA-ES 每次迭代找到的最佳參數與報酬率。這就是一個完美的 MLOps 閉環。