
# 使用 Anaconda Prompt 安裝套件 (Method 1)

這是在 Windows 上最穩定、最不會出錯的安裝方式，能確保環境變數正確，避免 PyCharm 終端機抓不到指令的問題。

### 步驟 1：開啟 Prompt

按 Windows 鍵，搜尋並開啟 "Anaconda Prompt (Miniconda3)"。

(注意：請勿使用 PowerShell 或一般的 CMD)

### 步驟 2：啟用環境

輸入以下指令切換到 PyCharm 建立的環境：

(這裡假設您的環境名稱為 py-tensorflow，若不確定可用 conda env list 查詢)



```Bash
conda activate py-tensorflow
```

### 步驟 3：執行安裝

確認提示字元最前面出現 `(py-tensorflow)` 後，直接貼上並執行以下指令：


**優先使用 Conda 安裝主要庫**：
盡量在同一次指令中安裝多個套件，這能讓 Conda 一次性解析所有相依性：
```powershell
conda install pandas scikit-learn pyarrow
```

**如果 Conda 找不到該套件，才使用 Pip**：
```powershell
pip install some-special-library

```

```Bash
pip install tensorflow pandas scikit-learn matplotlib seaborn pyarrow
```

#### 為什麼 TensorFlow 用 `pip` 安裝？

事實上，**TensorFlow 官方目前最推薦的安裝方式就是 `pip**`。

即便是在 Conda 環境中，Google 的 TensorFlow 團隊也建議使用 `pip install tensorflow`。原因如下：

1. **官方支援度**：Google 直接維護 PyPI (pip) 上的版本，更新最快且最穩定。
2. **效能優化**：`pip` 版本通常已經封裝好了對應的硬體加速（如 CUDA 支援，雖然 Windows 2024 年後主要是透過 WSL2 支援 GPU）。
3. **相容性**：Conda 頻道的 TensorFlow 有時是由第三方社群維護，版本更新較慢，且與部分 Python 套件可能會有路徑衝突。


### 步驟 4：完成

安裝完成後關閉視窗，回到 PyCharm。IDE 會自動偵測到變更並開始 **Indexing** (建立索引)，待右下角進度條跑完即可開始開發。