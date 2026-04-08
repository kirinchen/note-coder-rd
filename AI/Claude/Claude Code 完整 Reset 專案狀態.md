## 完整 Reset 專案狀態

### 1. 清除對話 context（當前 session）

在 Claude Code 內執行：

```
/clear
```

---

### 2. 清除所有 session 歷史紀錄

```bash
# macOS / Linux
rm -rf ~/.claude/projects/<你的專案路徑hash>/

# 或清除全部專案的 session
rm -rf ~/.claude/projects/
```

---

### 3. 清除 CLAUDE.md 記憶內容

```bash
# 專案層級
rm .claude/CLAUDE.md

# 或清空內容保留檔案
> .claude/CLAUDE.md
```

---

### 4. 清除專案 MCP 設定

```bash
rm .mcp.json
```

---

### 5. 清除 local scope 的 MCP / 設定

```bash
# 列出後逐一移除
claude mcp list
claude mcp remove <名稱>
```

---

### 一次執行全部（shell script）

```bash
#!/bin/bash
# 在專案根目錄執行

# 清除 CLAUDE.md
> .claude/CLAUDE.md

# 清除 .mcp.json（如果不需要保留）
# rm .mcp.json

# 清除該專案的 session 歷史
PROJECT_HASH=$(echo -n "$(pwd)" | md5sum | cut -d' ' -f1)
rm -rf ~/.claude/projects/${PROJECT_HASH}/

echo "✅ Claude Code 專案狀態已完整 reset"
```

---

### 重新進入就是全新狀態

```bash
cd your-project
claude
```

Claude 會像第一次進入這個專案一樣，沒有任何歷史記憶。