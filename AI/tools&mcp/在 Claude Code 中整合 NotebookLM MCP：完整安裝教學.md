# 在 Claude Code 中整合 NotebookLM MCP：完整安裝教學

> **適用對象：** 已安裝 Claude Code CLI，且有 Google NotebookLM 帳號的使用者  
> **工具版本：** notebooklm-mcp-cli v0.5.x（2026年4月最新）

---

## 什麼是 NotebookLM MCP？

[notebooklm-mcp-cli](https://github.com/jacob-bd/notebooklm-mcp-cli) 是一個開源工具，讓 Claude Code（以及 Cursor、Gemini CLI 等 AI 工具）能夠直接操控 Google NotebookLM。透過這個 MCP，你可以用自然語言指揮 Claude Code：

- 建立 / 管理 Notebook
- 匯入來源（URL、YouTube、Google Drive、文字）
- 查詢 Notebook 內容
- 生成 Audio Podcast、影片、簡報、Flashcard
- 分享 Notebook

> ⚠️ **免責聲明：** 此工具使用 NotebookLM 的非官方內部 API，可能隨時失效，僅供個人實驗使用。

---

## 前置需求

|需求|說明|
|---|---|
|Python 3.10+|確認：`python3 --version`|
|`uv` 套件管理工具|安裝：`curl -LsSf https://astral.sh/uv/install.sh \| sh`|
|Claude Code CLI|已安裝並登入|
|Google 帳號|有使用 NotebookLM 的帳號|

---

## 步驟一：安裝套件

推薦使用 `uv`（速度快、環境乾淨）：

```bash
uv tool install notebooklm-mcp-cli
```

安裝完成後，確認兩個執行檔都存在：

```bash
nlm --version
notebooklm-mcp --version
```

應看到類似輸出：

```
notebooklm-mcp-cli v0.5.26
```

---

## 步驟二：Google 帳號認證

MCP 需要取得你的 NotebookLM cookie 才能操作，執行登入指令：

```bash
nlm login
```

這會自動開啟一個瀏覽器視窗，完成 Google 登入後，cookie 會自動擷取並儲存。

**確認認證成功：**

```bash
nlm login --check
```

> **多帳號支援：** 如果你有多個 Google 帳號，可以用 profile 管理：
> 
> ```bash
> nlm login --profile work
> nlm login --profile personal
> nlm login profile list
> ```

---

## 步驟三：將 MCP 加入 Claude Code

有兩種方式，擇一即可。

### 方法 A：使用 `nlm setup`（最簡單，推薦）

```bash
nlm setup add claude-code
```

這個指令會自動偵測 Claude Code 的設定路徑並寫入，不需要手動編輯 JSON。

### 方法 B：手動用 `claude mcp add-json`

```bash
# User scope（所有專案都可用）
claude mcp add-json notebooklm-mcp \
  '{"command":"uvx","args":["--from","notebooklm-mcp-cli","notebooklm-mcp"]}' \
  --scope user

# 或 Project scope（只在目前專案）
claude mcp add-json notebooklm-mcp \
  '{"command":"uvx","args":["--from","notebooklm-mcp-cli","notebooklm-mcp"]}' \
  --scope project
```

**Scope 選擇建議：**

|Scope|使用時機|設定位置|
|---|---|---|
|`user`|個人常用工具，所有專案都要|`~/.claude.json`|
|`project`|只在特定專案用，可 commit|`./.mcp.json`|
|`local`|個人用、只在這個專案（預設）|`~/.claude.json`（含 project path）|

---

## 步驟四：確認連線

重新啟動 Claude Code，或在 session 內執行：

```
/mcp
```

看到 `notebooklm-mcp: connected` 即表示成功。

---

## 步驟五：開始使用

MCP 連線後，直接用自然語言跟 Claude Code 說話即可：

### 基本操作

```
列出我所有的 NotebookLM Notebooks
```

```
建立一個新的 Notebook，名稱叫「AI 研究筆記」
```

```
把這個 URL 加到我的 Notebook：https://example.com/article
```

### 查詢與分析

```
這個 Notebook 的主要內容是什麼？請幫我摘要
```

```
搜尋 Notebook 裡關於「機器學習」的部分
```

### 內容生成

```
幫我從這個 Notebook 生成一個 Podcast 音訊
```

```
製作一個簡報 Slide Deck，主題是這個 Notebook 的研究成果
```

```
建立 Flashcard，難度設中等
```

### 管理與分享

```
把這個 Notebook 設成公開連結
```

```
同步我 Google Drive 裡的來源，更新過期的文件
```

---

## 常見問題

### Q：認證過期怎麼辦？

Cookie 有效期約 2-4 週，失效時重新執行：

```bash
nlm login
```

MCP 本身在遇到認證錯誤時會嘗試自動重新整理 token，如果自動刷新失敗才需要手動重新登入。

### Q：升級到最新版本

```bash
# 如果 upgrade 沒裝到最新版，用 force 強制重裝
uv tool install --force notebooklm-mcp-cli
```

升級後在 Claude Code 內執行 `/mcp` 重新連線。

### Q：MCP 工具太多（35 個）佔用 context 怎麼辦？

不使用 NotebookLM 時記得關閉這個 MCP，在 Claude Code session 內：

```
@notebooklm-mcp
```

可以切換開關狀態。

### Q：診斷安裝問題

```bash
nlm doctor
```

這個指令會自動偵測安裝狀態、認證狀態、以及各 AI 工具的設定是否正確。

---

## 完整移除

```bash
# 移除套件
uv tool uninstall notebooklm-mcp-cli

# 從 Claude Code 移除設定
nlm setup remove claude-code

# 刪除認證資料（可選）
rm -rf ~/.notebooklm-mcp-cli
```

---

## 參考資源

- [GitHub 專案頁面](https://github.com/jacob-bd/notebooklm-mcp-cli)
- [MCP 完整工具清單（35 個工具）](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/MCP_GUIDE.md)
- [CLI 指令參考](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/CLI_GUIDE.md)
- [認證詳細說明](https://github.com/jacob-bd/notebooklm-mcp-cli/blob/main/docs/AUTHENTICATION.md)