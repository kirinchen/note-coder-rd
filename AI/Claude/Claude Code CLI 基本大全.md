
# Claude Code CLI 好用指令完全指南

> 整理最實用的 Claude Code CLI 指令，從基礎到進階，讓你的開發工作流更高效。

---

## 📦 安裝與登入

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows (PowerShell)
irm https://claude.ai/install.ps1 | iex

# 登入帳號
claude auth login

# 確認登入狀態
claude auth status

# 查看版本
claude --version

# 更新至最新版
claude update
````

---

## 🚀 啟動模式

```bash
# 進入互動式 REPL（最常用）
claude

# 直接帶 prompt 啟動
claude "幫我分析這個專案的結構"

# 非互動模式，執行後直接輸出結果（適合腳本 / CI）
claude -p "解釋這個 function"

# 繼續上一次的對話
claude --continue
claude -c

# 恢復特定 session（ID 可從 /cos 查詢）
claude --resume <session-id>
```

---

## ⌨️ Slash 指令（互動模式內使用）

| 指令         | 功能                              |
| ---------- | ------------------------------- |
| `/help`    | 顯示所有可用指令                        |
| `/clear`   | 清除對話 context（開始新任務前用）           |
| `/compact` | 壓縮對話歷史、釋放 context 空間            |
| `/exit`    | 離開 REPL                         |
| `/config`  | 開啟設定介面                          |
| `/model`   | 切換 AI 模型（Sonnet / Opus / Haiku） |
| `/mcp`     | 查看 MCP server 連線狀態              |
| `/context` | 查看目前 context 使用量                |
| `/review`  | 對目前修改進行 code review             |
| `/doctor`  | 健康檢查 Claude Code 安裝狀態           |
| `/cos`     | 顯示本次 session 的費用與時間             |
| `/agents`  | 管理 sub-agents                   |
| `/rewind`  | 時光機：回到對話的某個 checkpoint          |
| `/init`    | 在專案建立 `CLAUDE.md` 記憶檔           |

---

## ⌨️ 快捷鍵

|快捷鍵|功能|
|---|---|
|`Ctrl+C`|取消當前操作|
|`Ctrl+D`|離開 REPL|
|`Tab`|自動補全|
|`↑ / ↓`|瀏覽歷史指令|
|`#` + 文字|快速新增記憶到 CLAUDE.md|

---

## 🔧 MCP Server 管理

```bash
# 新增 HTTP remote MCP server
claude mcp add --transport http <名稱> <url>

# 新增 stdio MCP server（npm 套件）
claude mcp add --transport stdio <名稱> -- npx -y <套件名>

# 用 JSON 格式直接新增（適合複雜設定）
claude mcp add-json <名稱> '{"type":"http","url":"https://..."}'

# 列出所有 MCP server
claude mcp list

# 移除 MCP server
claude mcp remove <名稱>

# 加上 scope（影響範圍）
# user：所有專案都可用
claude mcp add --transport http --scope user wix-mcp-remote https://mcp.wix.com/mcp

# project：團隊共享（寫入 .mcp.json）
claude mcp add --scope project <名稱> ...
```

---

## 📂 Context 與檔案管理

```bash
# 指定額外目錄給 Claude 讀取（monorepo 常用）
claude --add-dir ../frontend ../backend

# 使用 @ 引用特定檔案（在對話中）
# 例如：
# > 幫我 review @./src/index.ts 這個檔案

# pipe 內容給 Claude 分析
cat error.log | claude -p "找出這個 log 的根本原因"
git log --oneline | claude -p "幫我整理這些 commit"
```

---

## 🛡️ 工具權限控制

```bash
# 只允許特定工具（更安全）
claude --allowedTools "Bash(git:*)" "Write" "Read"

# 禁止危險指令
claude --disallowedTools "Bash(rm:*)" "Bash(sudo:*)"

# 跳過所有權限提示（危險，CI 環境慎用）
claude --dangerously-skip-permissions
```

---

## 🤖 非互動模式（腳本 / CI 自動化）

```bash
# 基本非互動執行
claude -p "分析這個 codebase"

# 輸出成 JSON 格式
claude -p --output-format json "找出所有 TODO"

# 串流 JSON
claude -p --output-format stream-json "審查程式碼"

# 限制最多幾輪對話（節省 token）
claude -p --max-turns 3 "執行測試並回報結果"

# 加入自訂 system prompt
claude -p "分析錯誤" --append-system-prompt "你是一位 SRE 專家"

# 指定允許的工具（更精確控制）
claude -p "查詢資料" --allowedTools "Bash,Read,mcp__datadog"
```

---

## 📝 自訂 Slash 指令

在 `.claude/commands/` 建立 Markdown 檔案即可新增專案專屬指令：

```bash
mkdir -p .claude/commands

# 建立 /optimize 指令
echo "分析這段程式碼的效能問題並提出優化建議：" > .claude/commands/optimize.md

# 建立 /security 指令
echo "審查這段程式碼的安全漏洞：" > .claude/commands/security.md
```

個人全域指令（跨所有專案）放在 `~/.claude/commands/`：

```bash
mkdir -p ~/.claude/commands
echo "用繁體中文幫我 review 這個 PR 的改動：" > ~/.claude/commands/review-zh.md
```

---

## 🧠 CLAUDE.md 記憶系統

`CLAUDE.md` 是 Claude 讀取的專案記憶檔，每次啟動都會自動載入。

```bash
# 在專案根目錄初始化
/init

# 對話中快速新增記憶
# > # 這個專案使用 2 空格縮排，且不用分號
```

記憶檔位置優先順序：

```
.claude/CLAUDE.md          ← 當前專案（最優先）
~/.claude/CLAUDE.md        ← 個人全域設定
```

---

## 🔄 常見工作流程組合

```bash
# 分析 git 歷史並生成測試
claude -p "分析最近 10 個 commit" --output-format json > analysis.json
claude -p "根據分析結果生成測試" --max-turns 3

# 用 pipe 做 log 分析
tail -n 100 app.log | claude -p "找出異常 pattern 並給出解決建議"

# 讓 Claude 幫你寫 commit message
git diff --staged | claude -p "幫我寫一個符合 Conventional Commits 格式的 commit message"
```

---

## 📋 Scope 設定速查

|Scope|Flag|設定檔位置|適用情境|
|---|---|---|---|
|Local（預設）|無|`~/.claude.json`|個人、僅限當前專案|
|Project|`--scope project`|`.mcp.json`（可 git commit）|團隊共享設定|
|User / Global|`--scope user`|`~/.claude.json`|個人、所有專案通用|

---

## 💡 實用小技巧

- **`!` 前綴**：在對話中直接執行 shell 指令（跳過 Claude 的對話模式，省 token）
    
    ```
    > ! git status
    ```
    
- **`/compact`**：context 快滿時使用，壓縮歷史同時保留重要資訊
    
- **`/clear`**：切換任務時清除 context，避免前一個任務干擾
    
- **`#` 前綴**：對話中快速新增規則到 CLAUDE.md，例如：
    
    ```
    > # 所有 API 回應都要用 camelCase
    ```
    
- **選模型策略**：
    
    - Haiku → 快速探索、簡單問答
    - Sonnet → 日常開發（預設）
    - Opus → 複雜架構設計、安全分析

---

## 🔗 官方文件

- [Claude Code 文件](https://docs.claude.com/en/docs/claude-code/overview)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [MCP 整合指南](https://code.claude.com/docs/en/mcp)