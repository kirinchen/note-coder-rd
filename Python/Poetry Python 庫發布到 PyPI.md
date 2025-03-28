---
title: Poetry Python 庫發布到 PyPI
tags: [python]

---

# Poetry 將你的 Python 庫發布到 PyPI

要使用 Poetry 將你的 Python 庫發布到 PyPI，以下是具體的步驟。Poetry 提供了非常簡單的命令來管理這一過程，無需手動設置複雜的打包和發布流程。

### 步驟 1：確保 `pyproject.toml` 配置正確

首先，確保你的 `pyproject.toml` 文件包含必要的配置信息。這些信息將用於在 PyPI 上顯示你的包的基本資料。

#### `pyproject.toml` 範例：
```toml
[tool.poetry]
name = "your_lib"  # 替換為你的專案名稱
version = "0.1.0"  # 初次發布的版本號
description = "A simple common library"
authors = ["Your Name <your.email@example.com>"]
repository = "https://github.com/your-username/your-project"
license = "MIT"  # 替換為適當的 license

[tool.poetry.dependencies]
python = "^3.7"
requests = "^2.25.1"  # 列出你依賴的庫

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

### 步驟 2：設定 PyPI 的憑證

1. **建立 PyPI 帳號**：
   如果你還沒有 PyPI 帳號，請前往 [https://pypi.org/](https://pypi.org/) 註冊一個帳號。

2. **設定 Poetry 認證**：
   使用 Poetry 的 `poetry config` 命令來設定 PyPI 認證，這樣你可以安全地將包發布到 PyPI。

   執行以下命令來設定 PyPI 帳號和密碼：

   ```bash
   poetry config pypi-token.pypi <YOUR_PYPI_API_TOKEN>
   ```

   - 你可以從 PyPI 的 [API Token](https://pypi.org/manage/account/token/) 頁面生成一個新的 API Token。
   - 替換 `<YOUR_PYPI_API_TOKEN>` 為你從 PyPI 獲得的 API Token。

### 步驟 3：打包你的專案

在發布之前，先確保你的專案已經被正確打包。使用以下命令生成可發布的包：

```bash
poetry build
```

這會在 `dist/` 目錄下生成 `.tar.gz` 和 `.whl` 格式的包，這兩種格式都是 Python 包的標準發佈格式。

### 步驟 4：發布到 PyPI

使用 `poetry publish` 命令將包發布到 PyPI。

#### 如果已經設定好 API Token：
直接執行以下命令即可發布到 PyPI：

```bash
poetry publish --build
```

#### 如果沒有設定 API Token（手動輸入帳號與密碼）：
你可以直接運行以下命令，Poetry 會要求你輸入 PyPI 帳號和密碼：

```bash
poetry publish --build --username <your-pypi-username>
```

### 步驟 5：驗證發布

發布成功後，你可以前往 [https://pypi.org/](https://pypi.org/) 搜尋你的包並驗證其是否已成功上傳。如果已上傳，你應該能夠在 PyPI 上看到你的包信息，包括版本、說明、安裝指南等。

### 步驟 6：安裝測試

在你的包成功發布後，最好在其他環境中進行安裝測試，確保一切運作正常。你可以在新環境中執行：

```bash
pip install your_lib
```

這將從 PyPI 下載並安裝你發布的包。

### 總結
1. 確保 `pyproject.toml` 文件已正確配置。
2. 使用 `poetry config` 設定 PyPI 認證（推薦使用 API Token）。
3. 使用 `poetry build` 來打包你的專案。
4. 使用 `poetry publish --build` 來將你的包發布到 PyPI。

這樣，你的 Python 包就能被其他用戶通過 `pip install` 進行安裝和使用。


## 如果poetry找不到新版再 PyPI

```shell
poetry cache clear pypi --all
```
