# Meta-level ReAct × n8n 自動化架構筆記

tags: `claude-code` `n8n` `ReAct` `workflow-automation` `AI-architecture`

---

## 一、三種 Agent 執行模式比較

| | ReAct | Plan-and-Execute | 你的模式 |
|---|---|---|---|
| 規劃時機 | 執行時即時推理 | 執行時先規劃 | 設計時（Claude Code 預生成） |
| 靈活性 | 高 | 中 | 低（但刻意為之） |
| 每次執行成本 | 高 | 中 | 最低（幾乎無 LLM 呼叫） |
| 步驟可靠性 | 不穩定 | 依計畫品質 | 最穩定（固定流程） |
| 適合任務 | 探索性 | 複雜多步 | 重複性、生產級任務 |

---

## 二、你的架構定義

### 核心概念

這不是 ReAct + Plan-and-Execute 的組合，而是**第三種獨立架構**：

```
Claude Code（設計時）= Planner
n8n workflow（執行時）= Executor（確定性，非 Agent）
```

Plan-and-Execute 的 Planner 和 Executor 都在**執行時**運作；  
你的 Planner（Claude Code）完全在**設計時**完成，執行時的 n8n 不做任何推理。

可稱為：**"AI-assisted Workflow Engineering"** 或 **"Code Generation + Deterministic Execution"**

### FB 留言總結範例

```
Claude Code（一次性設計）
  └─→ 生成 n8n workflow JSON
        ├─ Schedule Trigger（每天定時）
        ├─ Facebook Graph API → 抓留言
        ├─ 過濾/清洗節點
        ├─ AI Summarize 節點（只在這裡用 LLM）
        └─ 輸出（Email / Slack / Notion）
```

### 核心優勢

- **可審查**：workflow JSON 可以 code review、版控
- **可預測**：每次執行路徑完全相同
- **省錢**：LLM 只在 Summarize 那一步用，其他節點是確定性邏輯
- **可維護**：出錯時 debug 容易，不像 Agent 的黑盒

---

## 三、加入 Feedback Loop → 完整架構

加入 n8n log → Claude Code Hook → 優化 後，系統升級為**自我修正系統**：

```
┌─────────────────── 設計時 ───────────────────┐
│  Claude Code → 生成 n8n workflow JSON        │
└──────────────────────────────────────────────┘
                    ↓ deploy
┌─────────────────── 執行時 ───────────────────┐
│  n8n 定時執行（確定性流程）                    │
│  FB抓留言 → 清洗 → Summarize → 輸出          │
└──────────────────────────────────────────────┘
                    ↓ fails / 邏輯落差
┌─────────────────── 優化時 ───────────────────┐
│  Claude Code Hook 收到 log                   │
│  分析問題 → 修改 workflow → redeploy         │
└──────────────────────────────────────────────┘
                    ↓ 循環
```

### Feedback Loop 與 ReAct 的對應關係

| ReAct 元素 | 你的架構 |
|---|---|
| Observation | n8n 執行結果 |
| Thought + Action | Claude Code Hook 分析優化 |
| Plan | 重新生成 workflow |

> **結論：你的 ReAct 運作在 Meta 層（workflow 本身），而不是 workflow 內部。**  
> 這是一種 **"Meta-level ReAct over Deterministic Plan-and-Execute Workflows"**，屬於 AIOps / Self-healing Automation 的前沿思路。

---

## 四、Claude Code 部署位置

### 建議：獨立在主機環境（不放進 docker-compose）

Claude Code 是**管理者角色**，讓管理者住在被管理的容器群裡，邏輯上倒置了。

| | 放進 docker-compose | 獨立在主機環境 |
|---|---|---|
| 存取 n8n API | 透過 internal network，安全 | 透過 localhost port，也 OK |
| 存取 PostgreSQL | 直接 internal，快 | 需開 5432 |
| Claude Code 更新自身 | ⚠️ 容器內改容器很麻煩 | ✅ 直接操作檔案系統 |
| Redeploy workflow | 需要 Docker socket 掛載（危險） | ✅ 直接呼叫 n8n API |
| 維護與 debug | 多一層容器複雜度 | ✅ 直接跑，log 清楚 |

### 推薦架構

```
主機
├─ Claude Code Hook（主機直接運行）
│   ├─ 監聽 n8n webhook / pg log
│   ├─ 呼叫 localhost:17835 (n8n API)
│   └─ 讀取 localhost:5432 (postgres)
│
└─ docker-compose（被管理的服務群）
    ├─ n8n
    ├─ n8n-worker
    ├─ postgres (pgvector)
    └─ redis
```

---

## 五、Spec 與 Workflow 的版控策略

### Single Source of Truth 三個層次

```
1. Spec / Prompt      → 告訴 Claude Code 要做什麼
2. n8n Workflow JSON  → Claude Code 生成的產出
3. 執行 log / 錯誤    → 觸發優化的輸入
```

### 建議架構（精簡版）

```
GitHub Repo
├─ specs/
│   └─ fb-daily-summary.md    ← 你寫的需求 spec
├─ workflows/
│   └─ fb-daily-summary.json  ← Claude Code 生成
└─ CLAUDE.md                  ← Claude Code 的全域規則
```

### 演進路徑

```
現在：Claude Code → n8n API（直接 deploy）
穩定：Claude Code → git commit → GitHub（備份）
未來：git push → GitHub Action → n8n API（完整 CI/CD）
```

> **現階段 GitHub Action 只需做格式驗證，不需要做 deploy。**  
> Spec 一定要進 GitHub（真相來源），workflow JSON 也要進（產出記錄），  
> 但 CI/CD deploy 讓 Claude Code 直接呼叫 n8n API 就夠了。

---

## 六、Claude Code 計費方式

| | API Key | Pro 訂閱（$20/月） |
|---|---|---|
| 使用方式 | ANTHROPIC_API_KEY 環境變數 | browser login / OAuth |
| 計費 | 按 token 用多少付多少 | 固定月費，共享額度 |
| 適合 | 自動化、CI/CD、非互動模式 | 互動式開發 |
| 額度限制 | 無（錢用完才停） | 有 5hr rolling window |

### 建議路線

- **互動式開發**（手動用 Claude Code）→ Pro 訂閱 $20/月 即可
- **自動化 Hook**（n8n log 觸發 → 自動優化）→ API Key，按用量計費（頻率低，費用極低）

> **構想階段：** 直接用 API Key + Pay-as-you-go，開 Console 拿 Key 就能跑，  
> 每次生成修改幾個 workflow，一個月可能連 $5 美金都不到。

---

## 七、是否需要 spec-kit？

[spec-kit](https://github.com/github/spec-kit) 是 GitHub 官方的 Spec-Driven Development 工具，流程為：

```
constitution → specify → clarify → plan → tasks → implement
```

### 結論：現階段不需要

| | spec-kit | 手寫 spec |
|---|---|---|
| 流程 | 固定 6 步儀式 | 你決定 |
| 產出 | 軟體程式碼 | n8n workflow JSON |
| 規格深度 | 用戶故事、API 規格、資料模型 | 觸發、邏輯、輸出即可 |
| 學習成本 | 需要熟悉 specify CLI | 直接寫 Markdown |
| 適合場景 | 建構應用程式 | 描述自動化流程 |

spec-kit 解決「需求不清楚時如何系統性釐清」的問題；  
但你的 workflow 需求本來就很具體，一頁 Markdown 就能讓 Claude Code 完全理解。

### 手寫 Spec 範例

```markdown
# FB 每日留言總結

## 目的
每天早上 9 點自動抓取前一天 FB 粉專留言，
用 Claude 總結關鍵議題，發到 Slack #daily-report

## 觸發
Schedule Trigger：每天 09:00 Asia/Taipei

## 資料來源
- Facebook Graph API
- Page ID: ${FB_PAGE_ID}
- 抓取範圍：過去 24 小時的留言

## 處理邏輯
1. 過濾掉隱藏留言與垃圾訊息
2. 超過 100 則留言時只取按讚數前 50 則
3. Claude Summarize：條列前 5 大議題 + 情緒傾向

## 輸出
- Slack Webhook → #daily-report
- 格式：標題 + 條列 + 原文連結

## 錯誤處理
- API 失敗：retry 3 次，仍失敗發 Slack 警告到 #alerts
- 零留言：跳過，不發報告
```

> **等到你要用 Claude Code 開發有後端 API + 前端介面的管理系統時，才是 spec-kit 發揮的地方。**

---

## 八、Quick Start 步驟

1. 申請 [Anthropic Console](https://console.anthropic.com/) 帳號（免費）
2. 取得 API Key，設定 `ANTHROPIC_API_KEY` 環境變數
3. 安裝 Claude Code CLI：`npm install -g @anthropic-ai/claude-code`
4. 建立 GitHub repo，結構如第五節
5. 寫第一個 `specs/your-workflow.md`
6. 用 Claude Code 生成 `workflows/your-workflow.json`
7. 呼叫 n8n API 直接 deploy，驗證執行
8. 確認穩定後再考慮加 CI/CD 與 Feedback Hook
