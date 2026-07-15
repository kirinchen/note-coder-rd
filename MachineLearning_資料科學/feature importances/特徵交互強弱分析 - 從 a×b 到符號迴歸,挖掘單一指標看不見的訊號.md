

# 【量化交易實戰】特徵交互強弱分析：從 a×b 到符號迴歸，挖掘單一指標看不見的訊號

### 前言：單一特徵的天花板

跑完 [[別再憑感覺寫策略！用 Random Forest 與 XGBoost 萃取「特徵重要性」]] 之後，你手上有了一張單一特徵的強弱排行。但市場訊號往往不是單一指標給的，而是**指標之間的關係**給的：

- 「量增」本身沒意義，「量增 **且** 價格在低檔」才有意義 → `volume_chg × (1 / price_percentile)`
- 「RSI 高」不一定超買，「RSI 高 **且** 波動率極低」才是盤整末端 → `rsi × (1 / atr)`
- 有些關係甚至是非線性的週期調製 → `momentum × cos(day_phase)`

這種「兩個平庸特徵相乘後變成強特徵」的現象，就是 **特徵交互 (Feature Interaction)**。這篇筆記回答兩個問題：

1. 怎麼**系統性地產生**交互特徵（a×b、a÷b、a^b、a×cos(b)…）？
2. 怎麼**量化**這些組合出來的新特徵，到底比原始特徵強還是弱？

### 樹模型不是已經會學交互了嗎？

會，但很貴。Random Forest / XGBoost 靠「先切 a 再切 b」的巢狀分裂來逼近 a×b 的效果，這需要**夠深的樹 + 夠多的資料**才能拼出來，而且對除法、比值、週期函數這類關係的逼近效率很差。

**把交互項顯式地做成一欄餵給模型**，等於幫模型把答案的形狀先畫好，它只要學「用不用」而不是「怎麼拼」。這也是為什麼交互特徵做得好，淺層模型也能打贏深層模型。

### 第一步：手動建構交互項

#### 用 PolynomialFeatures 暴力展開乘積項

```Python
import pandas as pd
from sklearn.preprocessing import PolynomialFeatures

# 挑幾個你懷疑有交互效果的核心特徵（不要全丟，理由見後面的陷阱）
core = df[['rsi', 'volume_chg', 'atr', 'momentum']]

# degree=2 會產生所有兩兩乘積 (a*b) 與平方項 (a^2)
poly = PolynomialFeatures(degree=2, interaction_only=False, include_bias=False)
poly_arr = poly.fit_transform(core)
poly_df = pd.DataFrame(
    poly_arr,
    columns=poly.get_feature_names_out(core.columns),
    index=core.index,
)
print(poly_df.columns.tolist())
# ['rsi', 'volume_chg', ..., 'rsi volume_chg', 'rsi atr', ..., 'atr^2', ...]
```

#### 手工打造領域知識組合（比值、次方、週期）

`PolynomialFeatures` 只會乘法，除法、次方、三角函數要自己來——而這些往往才是有金融意義的：

```Python
import numpy as np

eps = 1e-9  # 防除零

fx = pd.DataFrame(index=df.index)
fx['vol_per_atr']    = df['volume_chg'] / (df['atr'] + eps)       # a/b：單位波動的量能
fx['rsi_sq']         = df['rsi'] ** 2                              # a^2：放大極端區
fx['mom_pow_vol']    = np.sign(df['momentum']) * (
                          df['momentum'].abs() ** df['atr'].clip(0.5, 2)
                       )                                           # a^b：波動率調製動能
fx['mom_cos_phase']  = df['momentum'] * np.cos(
                          2 * np.pi * df['bar_of_day'] / 288
                       )                                           # a*cos(b)：日內週期調製
fx['spread_ratio']   = (df['ma_5'] - df['ma_20']) / (df['atr'] + eps)  # 標準化乖離
```

> **注意 a^b 的定義域**：底數為負、指數非整數會產生 NaN/複數，務必先 `abs()` 加 `sign()` 拆開處理，或把指數 `clip` 在安全範圍。

### 第二步：讓模型評判交互項的強弱

把「原始特徵 + 交互特徵」**同場競技**，用和單一特徵完全相同的流程量強弱——impurity 粗篩、permutation 複核：

```Python
from sklearn.ensemble import RandomForestClassifier
from sklearn.inspection import permutation_importance
from sklearn.model_selection import train_test_split

X_all = pd.concat([core, fx], axis=1).replace([np.inf, -np.inf], np.nan).dropna()
y_all = y.loc[X_all.index]

# 時間序列不 shuffle
X_train, X_test, y_train, y_test = train_test_split(
    X_all, y_all, test_size=0.3, shuffle=False
)

model = RandomForestClassifier(n_estimators=300, random_state=42)
model.fit(X_train, y_train)

perm = permutation_importance(
    model, X_test, y_test, n_repeats=10, random_state=42
)
rank = pd.DataFrame({
    'feature': X_all.columns,
    'perm_mean': perm.importances_mean,
    'perm_std': perm.importances_std,
}).sort_values('perm_mean', ascending=False)

print(rank)
```

**判讀方式：**

| 觀察到的現象 | 解讀 |
|---|---|
| 交互項排名 **高於** 它的兩個母特徵 | 真交互：關係本身有訊號，母特徵可考慮退役 |
| 交互項排名高，但母特徵也高 | 可能只是共線性搭便車，做下面的「消融測試」確認 |
| 交互項 ≈ 0 | 這個組合是你的一廂情願，砍掉 |

#### 消融測試 (Ablation)：交互項的終極審判

排行榜會被共線性干擾（交互項天生和母特徵高度相關）。最乾淨的驗證是**拔掉再比**：

```Python
from sklearn.model_selection import TimeSeriesSplit, cross_val_score

def cv_score(X):
    cv = TimeSeriesSplit(n_splits=5)
    m = RandomForestClassifier(n_estimators=300, random_state=42)
    return cross_val_score(m, X, y_all, cv=cv, scoring='accuracy').mean()

base   = cv_score(X_all.drop(columns=['vol_per_atr']))
with_f = cv_score(X_all)
print(f"不含交互項: {base:.4f} → 含交互項: {with_f:.4f}")
```

分數有實質提升，這個交互項才算真的通過審判——排行榜分數再高都只是候選人。

### 第三步：用符號迴歸自動搜尋公式（gplearn）

手動組合終究受限於想像力。**符號迴歸 (Symbolic Regression)** 用遺傳規劃 (GP) 在「運算子空間」裡演化公式：隨機產生一堆 `mul(rsi, cos(atr))` 這種語法樹，留下適應度高的，交配、突變、再淘汰——等於讓演算法幫你窮舉 a×b、a÷b、a×cos(b) 的所有變形。

```Python
from gplearn.genetic import SymbolicTransformer

st = SymbolicTransformer(
    generations=20,
    population_size=2000,
    hall_of_fame=100,
    n_components=10,            # 最終輸出 10 條演化出的公式特徵
    function_set=('add', 'sub', 'mul', 'div', 'sqrt', 'log', 'abs', 'sin', 'cos'),
    parsimony_coefficient=0.001, # 懲罰過長公式，抑制過擬合
    max_samples=0.8,
    random_state=42,
)
st.fit(X_train, y_train)

# 看演化出了什麼公式
for prog in st:
    print(prog)
# 例：mul(div(volume_chg, atr), cos(momentum))

gp_train = pd.DataFrame(st.transform(X_train), index=X_train.index,
                        columns=[f'gp_{i}' for i in range(10)])
gp_test  = pd.DataFrame(st.transform(X_test), index=X_test.index,
                        columns=[f'gp_{i}' for i in range(10)])
```

演化出的 `gp_*` 特徵一樣要丟回第二步的 permutation + 消融流程審判，**gplearn 的適應度是在訓練集上算的，它挑出來的公式天生就有過擬合傾向**。

### 陷阱與防雷

1. **組合爆炸**：20 個特徵取兩兩乘積就是 190 欄，再加除法、次方直接破千。特徵越多、噪音越多、算力越貴。**先用單一特徵排行選出前 5～10 名再做交互**（可搭配 [[AutoML-03 巨量特徵的殘酷淘汰賽 (Massive Feature Selection)]] 的流程）。
2. **多重比較謬誤**：你測 1000 個隨機組合，就算全是噪音，也**必然**有幾個在回測期表現亮眼。組合測得越多，對「亮眼」的懷疑要越深——務必留一段從未參與篩選的 hold-out 期做最終驗證。
3. **交互項與母特徵的共線性**：a×b 和 a、b 本來就高度相關，permutation importance 會互相稀釋（母特徵幫交互項補位）。這正是消融測試不可省略的原因。
4. **量綱與尺度**：不同單位的特徵直接相乘，數值範圍可能橫跨十幾個數量級，除法遇到近零母數會噴出極端值。先標準化、加 `eps`、`replace([inf, -inf], nan)` 是基本衛生。
5. **可解釋性斷崖**：`mul(div(a, sqrt(b)), cos(c))` 就算分數高，你說不出它的市場意義時，它在制度改變（regime change）後失效的機率遠高於有經濟邏輯的組合。**看得懂的 a×b 優先於看不懂的演化公式。**

### 結論

- 交互特徵的價值在於**把「關係」顯式化**，讓模型不必用巢狀分裂硬拼。
- 產生方式三層遞進：`PolynomialFeatures` 暴力乘積 → 領域知識手工組合（比值、次方、週期）→ gplearn 符號迴歸自動演化。
- 強弱審判和單一特徵同一套：impurity 粗篩 → permutation 複核 → **消融測試定生死**。
- 記住兩條鐵律：**先篩後乘**（控制組合爆炸）、**留 hold-out**（防多重比較謬誤）。
