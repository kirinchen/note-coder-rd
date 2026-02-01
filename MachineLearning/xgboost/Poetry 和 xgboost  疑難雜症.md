## **XGBoost 檔案太大下載太久**
（通常編譯好的 Wheel 檔約 **100MB ~ 300MB**），如果您連線到官方 PyPI 伺服器（通常在美國），加上跨海纜線壅塞，下載真的會等到天荒地老。

針對您的疑問：

1. **XGBoost 很大嗎？** 是的，因為它包含大量 **C++ 編譯後的底層運算庫**，有些版本還包進了 NVIDIA 的 GPU 函式庫，所以不像純 Python 套件只有幾 KB。

    

解決「下載超久」的正確解法不是換工具，而是**換下載源（Mirror）**。

### 🚀 極速解決方案：切換到台灣/亞洲鏡像站

我們可以告訴 Poetry：「不要去美國下載，改去台灣的國網中心（NCHC）或 Google 的鏡像站下載」。這通常能讓下載速度提升 10 倍以上。

請修改您的 `pyproject.toml`，在檔案最下方加入這段設定：

#### 方法 A：修改 `pyproject.toml` (推薦，專案級別生效)

直接把這段貼到您的 `pyproject.toml` 最後面：



```TOML
[[tool.poetry.source]]
name = "google"
url = "https://pypi.org/simple" # Google Cloud CDN, 通常很快且穩定
priority = "primary"
```

或者使用台灣國網中心 (NCHC)：



```TOML
[[tool.poetry.source]]
name = "nchc"
url = "https://free.nchc.org.tw/pypi/simple"
priority = "primary"
```

存檔後，再次執行：



```Bash
# 清除快取並重新 lock (確保它抓到新來源)
poetry lock --no-update
poetry install
```

---
