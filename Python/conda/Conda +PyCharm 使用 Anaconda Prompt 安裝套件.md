
# 使用 Anaconda Prompt 安裝套件 (Method 1)

這是在 Windows 上最穩定、最不會出錯的安裝方式，能確保環境變數正確，避免 PyCharm 終端機抓不到指令的問題。

### 步驟 1：開啟 Prompt

按 Windows 鍵，搜尋並開啟 "Anaconda Prompt (Miniconda3)"。

(注意：請勿使用 PowerShell 或一般的 CMD)

### 步驟 2：啟用環境

輸入以下指令切換到 PyCharm 建立的環境：

(這裡假設您的環境名稱為 py-tensorflow，若不確定可用 conda env list 查詢)

Bash

```
conda activate py-tensorflow
```

### 步驟 3：執行安裝

確認提示字元最前面出現 `(py-tensorflow)` 後，直接貼上並執行以下指令：

Bash

```
pip install tensorflow pandas scikit-learn matplotlib seaborn pyarrow
```

### 步驟 4：完成

安裝完成後關閉視窗，回到 PyCharm。IDE 會自動偵測到變更並開始 **Indexing** (建立索引)，待右下角進度條跑完即可開始開發。