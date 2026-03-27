這是一份為您量身打造的**「人工智慧與機器學習全景演算法 Roadmap (地圖)」**。

我將以 AI 發展史上的**「三大哲學陣營」**為核心骨幹，並為了保證現代機器學習的完整性，加入了第四個板塊**「統計與類推學派」**。這樣不僅能將您熟悉的最佳化演算法（CMA-ES、GA）、量化常用的模型（XGBoost、KNN），以及您點名的 Q-Learning 全部收錄，還能讓您一眼看透它們的血緣關係！

---

### 🗺️ AI 與機器學習全景 Tree Map

 [![, AI generated](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcSsYxnaBT02C9vM2Oy0W3w-38d_ubtq_v6mV3eusmwbDCli_Fo-_Ffwr1hKxTEM54zF3Jkyv0sNAJkVyYotm5E7L-38czQmvcJWLosFbgZhotcwXmY) Opens in a new window](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcSsYxnaBT02C9vM2Oy0W3w-38d_ubtq_v6mV3eusmwbDCli_Fo-_Ffwr1hKxTEM54zF3Jkyv0sNAJkVyYotm5E7L-38czQmvcJWLosFbgZhotcwXmY)

Shutterstock

Explore



```Plaintext
🧠 人工智慧與機器學習 (AI & ML)
│
├── 🏛️ 1. 符號主義 (Symbolism) —— 「智慧來自邏輯推演與符號規則」
│   │
│   ├── 傳統精確演算法 (Exact Algorithms)
│   │   ├── 動態規劃 (Dynamic Programming)
│   │   └── 貪婪演算法 (Greedy Algorithm)
│   │
│   ├── 專家系統 (Expert Systems)
│   │   └── 知識庫與推論引擎 (Knowledge Base & Inference Engine)
│   │
│   └── 符號與資訊熵學習 (Symbolic / Information-based Learning)
│       ├── 符號迴歸 (Symbolic Regression) -> e.g., gplearn
│       ├── 決策樹 (Decision Tree) -> CART, C4.5
│       └── 🏆 樹狀集成模型 (Tree Ensembles) -> Random Forest, XGBoost, LightGBM
│
├── 🕸️ 2. 連結主義 (Connectionism) —— 「智慧來自神經元的仿生連結與權重」
│   │
│   ├── 早期類神經網路 (Artificial Neural Networks, ANN)
│   │   └── 多層感知器 (MLP)
│   │
│   ├── 拓樸與無監督網路 (Topological & Unsupervised Networks)
│   │   └── 🗺️ 自組織地圖 (SOM - Self-Organizing Map)
│   │
│   └── 深度學習 (Deep Learning)
│       ├── 卷積神經網路 (CNN) -> 視覺、圖像辨識 (ResNet, YOLO)
│       ├── 循環神經網路 (RNN/LSTM) -> 傳統時間序列預測
│       └── 注意力機制與大模型 (Attention & Transformer) -> GPT-4, Claude
│
├── 🐜 3. 行為/進化主義 (Behaviorism) —— 「智慧來自與環境的互動、試錯與適者生存」
│   │
│   ├── 🎯 強化學習 (Reinforcement Learning) -> AI 玩遊戲、機器人控制
│   │   ├── 無模型學習 (Model-free) -> 🎮 Q-Learning, DQN
│   │   └── 策略梯度 (Policy Gradient) -> PPO, Actor-Critic (AlphaGo 的底層)
│   │
│   └── 🧬 元啟發式與演化計算 (Metaheuristics & Evolutionary) -> 您的主場！
│       ├── 軌跡型搜尋 (Trajectory) -> 模擬退火 (SA), 禁忌搜尋 (Tabu Search)
│       ├── 群體智能 (Swarm Intelligence) -> 蟻群 (ACO), 粒子群 (PSO), 獵鷹群 (HHO)
│       ├── 演化算法 (Evolutionary) -> 基因演算法 (GA), 遺傳規劃 (GP)
│       └── 連續空間頂級演化 -> 🏆 CMA-ES
│
└── 📊 4. 統計與類推學派 (Statistical & Analogizers) —— 「智慧來自機率分佈與歷史相似性」
    │
    ├── 基於實例與距離 (Instance-based / Distance)
    │   ├── 🔍 K-近鄰演算法 (KNN)
    │   └── 支持向量機 (SVM)
    │
    ├── 貝氏與機率學習 (Bayesian / Probabilistic)
    │   ├── 樸素貝氏 (Naive Bayes)
    │   └── 高斯混合模型 (GMM)
    │
    └── 無監督分群與降維 (Unsupervised Clustering & Dimensionality Reduction)
        ├── K-平均演算法 (K-Means)
        └── 主成分分析 (PCA)
```

---

### 📖 各大陣營與核心演算法說明

#### 🏛️ 1. 符號主義 (Symbolism)

**哲學：** 認為人類的思考就是符號的運算，只要能把世界抽象成 `If-Then` 的規則，機器就能思考。

- **XGBoost / Random Forest**：雖然它們被歸類在現代機器學習，但它們的靈魂是「符號主義」。一棵決策樹的本質，就是機器幫您從資料中找出一連串完美的 `If-Else` 規則（例如 `gapSize > 1` 且 `sdWidth < 0.5`）。這也是為什麼它們的結果人類完全看得懂（可解釋性極高）。
    
- **應用場景**：處理表格型資料（Tabular Data）、量化交易的技術指標組合。
    

#### 🕸️ 2. 連結主義 (Connectionism)

**哲學：** 反對手寫規則。主張只要給機器足夠多的「模擬神經元（節點）」和資料，神經元之間的「權重（Weights）」就會自己調整，智慧就會自然湧現。

- **SOM (自組織地圖)**：這是一種特殊的非監督式神經網路。它不像深度學習去預測明天會不會漲，而是把高維度的複雜特徵，映射並壓縮成一張 2D 地圖，把長得像的歷史時刻「聚落」在一起。
    
- **Transformer (GPT)**：目前的 AI 霸主。透過「注意力機制（Attention）」來模擬大腦在看句子時，焦點該放在哪個詞上面。
    
- **應用場景**：影像辨識、自然語言處理、把複雜市場狀態降維成 2D 視覺地圖。
    

#### 🐜 3. 行為 / 進化主義 (Behaviorism / Evolutionism)

**哲學：** 智慧不需要邏輯，也不需要巨大的腦容量。智慧是「在環境中跌倒、試錯、被淘汰後存活下來的經驗」。

- **強化學習 (Q-Learning)**：
    
    - **原理**：這就像訓練小狗。AI 是一個代理人 (Agent)，市場是環境。AI 隨便做一個動作（買入），如果賺錢就給獎勵（Reward），賠錢就扣分。**Q-Learning** 會在背後建立一張巨大的 `Q-Table`，記錄「在什麼狀態下，做什麼動作未來的總獎勵最高」。
        
    - **量化應用**：不預測漲跌，直接讓 AI 決定現在要「做多、做空還是空手」，用帳戶的最終 PnL 當作 Reward 讓它自己進化。
        
- **CMA-ES / GA (演化計算)**：
    
    - **原理**：模擬達爾文進化論。GA 用 0 和 1 模擬基因交配；**CMA-ES** 則是透過「高斯機率雲」在多維連續空間中變形，尋找讓存活率（勝率）最高的完美浮點數參數。
        
    - **量化應用**：用來為您的自訂函數（如 `WaveFunc`）尋找完美的數學參數，或是取代 Optuna 做超參數尋寶。
        

#### 📊 4. 統計與類推學派 (Statistical & Analogizers)

**哲學：** 認為世界上沒有絕對的規則，只有「機率」與「相似性」。歷史總是驚人地相似。

- **KNN (K-近鄰演算法)**：
    
    - **原理**：最極致的「類推」思想。不訓練任何模型規則，直接拿現在的 K 線特徵，去歷史資料庫算「歐式距離」，找出長得最像的 5 段歷史，這 5 段歷史的結果是什麼，現在就是什麼。
        
- **K-Means**：
    
    - **原理**：讓資料在空間中尋找自己的同伴，自動把市場分成 K 種不同的劇本（例如盤整區、主升段）。
        

---
