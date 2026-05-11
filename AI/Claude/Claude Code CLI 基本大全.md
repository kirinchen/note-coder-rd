
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

## 🎯 `-p` (Print Mode) 進階用法

`-p` / `--print` 是讓 Claude Code 跑「**一次性、非互動、印出結果就結束**」的核心 flag，幾乎所有自動化、CI、Hook、n8n 整合都依賴它。

### 基本語法

```bash
# 形式 1：直接帶 prompt
claude -p "你要做的事"

# 形式 2：從 stdin 讀入 prompt
echo "你要做的事" | claude -p

# 形式 3：prompt + pipe 內容當資料
cat file.log | claude -p "分析這份 log 找出錯誤"
```

### 常用組合 flag

| Flag | 作用 | 範例 |
|---|---|---|
| `--output-format text` | 純文字輸出（預設） | `claude -p "..." --output-format text` |
| `--output-format json` | 整包 JSON（含 metadata、cost） | `claude -p "..." --output-format json` |
| `--output-format stream-json` | 串流式 JSON，每個事件一行 | 適合即時處理 |
| `--max-turns N` | 限制最多幾輪工具呼叫 | `--max-turns 3` 防失控 |
| `--model <id>` | 指定模型 | `--model claude-opus-4-7` |
| `--append-system-prompt` | 在系統提示後追加內容 | 注入專家角色 |
| `--allowedTools` | 白名單工具 | `--allowedTools "Read,Bash(git:*)"` |
| `--disallowedTools` | 黑名單工具 | `--disallowedTools "Bash(rm:*)"` |
| `--add-dir` | 加入額外可讀取目錄 | monorepo 必備 |
| `--resume <id>` | 在 `-p` 模式下接續特定 session | 跨次累積 context |

### 實戰範例

```bash
# 1. CI 自動 Code Review（吐 JSON 給後續流程解析）
git diff origin/main | claude -p "review 這個 diff，列出潛在問題" \
  --output-format json \
  --max-turns 1 > review.json

# 2. 寫 commit message（吐純文字直接給 git）
git commit -m "$(git diff --staged | claude -p '寫一個 Conventional Commits 格式的訊息' --max-turns 1)"

# 3. n8n 觸發：分析錯誤 log
cat error.log | claude -p "找出根本原因並建議修法" \
  --append-system-prompt "你是一位 SRE 專家，回應請繁中、條列、附行號"

# 4. 跨次累積（先建脈絡、後續沿用）
SESSION_ID=$(claude -p "讀完 ./docs 後幫我記住專案" --output-format json | jq -r .session_id)
claude -p --resume "$SESSION_ID" "根據剛剛的記憶，寫 README"

# 5. 串流模式（即時把 token 餵給前端 / Slack）
claude -p "寫一份 incident 報告" --output-format stream-json | while read line; do
  echo "$line" | jq -r '.delta.text // empty'
done
```

### Exit Code

- `0`：成功
- `1`：一般錯誤（prompt 解析失敗、API 錯誤）
- `非 0`：流程腳本可直接用 `set -e` 或 `if !` 來判斷成敗

### 注意事項

- `-p` 模式**不會**讀互動模式的 `/init`、`/clear` 等指令——所有設定要靠 flag 帶入
- 預設仍會走權限提示，**腳本場景需搭配 `--allowedTools` 或 `--dangerously-skip-permissions`**
- `--max-turns` 強烈建議設，避免 LLM 自我迴圈燒 token
- stdin 同時有 prompt 與 pipe 資料時，**prompt 走參數、資料走 pipe**，別混在一起

---

## ⚠️ `--dangerously-skip-permissions` 深入解析

簡稱 **YOLO 模式**——跳過所有工具權限提示，Claude 可以直接執行 Bash、寫檔、改 git 等所有操作。**強大但危險**，下面說明何時用、何時不能用。

### 語法

```bash
# 互動模式
claude --dangerously-skip-permissions

# 非互動 + 跳權限（CI / Hook 的常見組合）
claude -p "..." --dangerously-skip-permissions
```

### 適合使用的情境 ✅

1. **隔離環境內**：Docker container、VM、worktree、Codespaces 等可隨時銷毀的 sandbox
2. **CI / GitHub Actions**：runner 是一次性容器，跑完即銷毀
3. **n8n Hook 自動優化 workflow**：背景 agent，需要無人值守地完成多步任務
4. **本機受信目錄 + 縮限工具**：搭配 `--allowedTools` 把可用工具收斂，例如只允許 `Read,Write,Bash(git:*)`

### 不建議使用的情境 ❌

- **直接在主機 `~/` 或專案根目錄跑**：一個 `rm -rf` 就毀了
- **連線到正式環境 DB / Server 的環境**
- **未經審查的 prompt**（例如從外部 webhook 直接餵 prompt 給 Claude）—— prompt injection 風險極高
- **共用主機 / 多用戶機器**

### 安全使用模式

```bash
# 模式 A：縮限工具 + 跳權限（推薦）
claude -p "重構 src/ 內的型別" \
  --allowedTools "Read,Edit,Write" \
  --disallowedTools "Bash" \
  --dangerously-skip-permissions

# 模式 B：開新 git worktree 隔離
git worktree add /tmp/yolo-branch HEAD
cd /tmp/yolo-branch
claude --dangerously-skip-permissions
# 完事後 review → 合回主分支 → 銷毀 worktree
git worktree remove /tmp/yolo-branch

# 模式 C：Docker 隔離（最安全）
docker run --rm -v $(pwd):/work -w /work node:20 \
  bash -c "npm i -g @anthropic-ai/claude-code && \
           claude -p '$PROMPT' --dangerously-skip-permissions"

# 模式 D：n8n Hook（背景優化 workflow）
claude -p "$ERROR_LOG → 修正 workflow JSON" \
  --allowedTools "Read,Write,Bash(curl:*)" \
  --dangerously-skip-permissions \
  --max-turns 5 \
  --output-format json
```

### 與其他權限機制的對比

| 方法 | 安全性 | 自動化適用度 | 說明 |
|---|---|---|---|
| 預設（每次提示） | 🟢 高 | 🔴 不可用 | 互動開發用 |
| `--allowedTools` | 🟢 高 | 🟡 可，但仍有部分提示 | 縮限工具範圍 |
| `--dangerously-skip-permissions` | 🔴 低 | 🟢 完全自動 | 跳過所有提示 |
| `allowedTools` + `skip-permissions` 合用 | 🟡 中 | 🟢 完全自動 | **推薦組合** |

### 一句話總結

> `-p` 讓 Claude 變腳本，`--dangerously-skip-permissions` 讓腳本不被打斷——兩者合用是 AI 自動化的基本套路，但**一定要配合隔離環境或工具白名單**，否則只是把生產環境的鑰匙交給 LLM。

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

# 開新 Session

直接用這個指令開新 session：

```bash
claude
```

在已有的 session 裡，你也可以用：

```
/new
```

---

**其他相關指令：**

|指令|說明|
|---|---|
|`claude`|開新 session|
|`/new`|在目前 session 內開新的|
|`/clear`|清除目前 conversation history（同 session，但記憶清空）|
|`/rename <name>`|幫目前 session 命名|

如果你是用 tmux 跑多個 session，可以開新 window 再跑 `claude`，這樣兩個 session 可以並行。

## 🔗 官方文件

- [Claude Code 文件](https://docs.claude.com/en/docs/claude-code/overview)
- [CLI Reference](https://code.claude.com/docs/en/cli-reference)
- [MCP 整合指南](https://code.claude.com/docs/en/mcp)