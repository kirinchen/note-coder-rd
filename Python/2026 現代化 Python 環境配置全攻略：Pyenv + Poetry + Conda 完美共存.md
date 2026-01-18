# 【RD 必備】2026 現代化 Python 環境配置全攻略：Pyenv + Poetry + Conda 完美共存

身為 RD，你的開發需求通常很多元：

1. **後端開發**：需要 Poetry 來處理 `pyproject.toml`，確保依賴版本精確。
2. **資料科學**：需要 TensorFlow / PyTorch，這些套件有時用 `conda` 安裝比 `pip` 穩定得多。
3. **多版本並行**：專案 A 用 Python 3.10，專案 B 用 3.12。

為了達成這些目標，我們的核心策略是：**讓 `pyenv` 成為唯一的「版本管理總管」**。

---

### 環境架構概覽

在進入安裝前，先理解這個層級關係。`pyenv` 位於最底層，負責提供不同版本的 Python「原材料」給上層工具。

---

### 第一步：安裝 Pyenv (系統總管)

#### 1. Linux / WSL 2 環境

Linux 是最友善的開發環境。如果你在 Windows 上，強烈建議使用 **WSL 2 (Ubuntu)**。

* **安裝編譯依賴** (極重要，否則安裝 Python 會報錯)：
```bash
sudo apt update
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git

```


* **使用官方腳本安裝**：
```bash
curl https://pyenv.run | bash

```


* **設定 Shell (將以下加入 `~/.zshrc` 或 `~/.bashrc`)**：
```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

```



#### 2. Windows 原生環境 (pyenv-win)

如果你必須在原生 Windows 下開發，請使用 `pyenv-win`。

* **使用 PowerShell 自動安裝**：
以管理員權限開啟 PowerShell 並執行：
```powershell
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"

```


* **檢查環境變數**：
確保 `USER` 環境變數中有 `PYENV` 和 `PYENV_ROOT`，且 `Path` 中最上方有 `%PYENV_ROOT%\bin` 和 `%PYENV_ROOT%\shims`。

---

### 第二步：安裝 Python 與 Anaconda

現在你可以透過 `pyenv` 安裝任何東西。

1. **安裝一般 Python (給 Poetry 用)**：
```bash
pyenv install 3.11.10
pyenv global 3.11.10  # 設定系統預設版本

```


2. **安裝 Anaconda/Miniconda (給 Data Science 用)**：
```bash
# 查看有哪些 conda 版本可選
pyenv install --list | grep miniconda

# 安裝最新版 Miniconda
pyenv install miniconda3-latest

```



---

### 第三步：相容性實戰 (Poetry vs. Conda)

這就是身為 RD 最關鍵的時刻：如何根據專案性質切換。

#### 模式 A：一般後端專案 (使用 Poetry)

對於 FastAPI、Django 等專案，我們使用 Poetry 管理。

1. 進入專案資料夾：`cd my-backend-project`
2. 指定 Python 版本：`pyenv local 3.11.10`
3. **讓 Poetry 建立環境**：
```bash
poetry init
poetry env use $(pyenv which python)
poetry add fastapi

```


*這時，Poetry 會使用 pyenv 提供的 Python 3.11 建立一個乾淨的虛擬環境。*

#### 模式 B：深度學習專案 (使用 Conda)

當你需要 CUDA 或特定的資料科學套件時：

1. 進入專案資料夾：`cd my-ds-project`
2. **切換到 Conda 環境**：`pyenv local miniconda3-latest`
3. **使用 Conda 建立環境**：
```bash
conda create -n tf-gpu python=3.10 tensorflow-gpu
conda activate tf-gpu

```


*由於你已經透過 `pyenv local` 切換了環境，你的 `conda` 指令現在是可用的，且不會與 Poetry 的環境衝突。*

---

### RD 的私藏小技巧

> **為什麼不直接裝 Anaconda？**
> Anaconda 會強行接管你的 `python` 指令，這會導致 Poetry 在解析依賴時變得非常緩慢且混亂。透過 `pyenv` 將 Anaconda「隔離」成其中一個 Python 版本，是你維持系統乾淨的最高準則。

---

### 結語

透過這套配置，你的電腦現在擁有了：

* **隔離性**：各專案互不干擾。
* **靈活性**：隨時切換純 Python 或 Conda。
* **專業度**：符合現代開發規範。

---

**想知道如何進一步在 VS Code 中設定讓它自動偵測這些 pyenv 環境嗎？**
