# 三個 Surface,一套 Engine:Multi-AI Agent 架構的角色分工設計

> 寫給:已經知道「要做 multi-agent」,但卡在「公司有 PM、有行政、有工程、有分析師,大家用的東西不一樣,怎麼統一?」的 IT / Tech Lead。
>
> TL;DR — Anthropic 給你的不是一個產品,是一個 surface 矩陣(CLI / Desktop App 的 Code tab / Cowork tab / Chat tab)。底層是同一個 Agent SDK 引擎、同一份 `CLAUDE.md`、同一套 plugin 與 MCP。架構設計的關鍵不是強迫全員用同一個 surface,而是**讓不同角色挑對的 surface,然後用 plugin 把行為規範統一在所有 surface 上**。

---

## 一、起點:你的公司不是清一色工程師

很多 multi-agent 架構的討論預設了一個 monolithic 的工程團隊 —— 全員 terminal 熟練、全員寫 code、全員能接受裝 plugin 改 hook。但真實的公司不長這樣:

- **PM、Operations、行政**:他們的工作是文件處理、跨工具流程、會議總結、報告產出。給他們 terminal 是錯的。
- **法務、財務、分析師**:他們處理合約、報表、SQL 結果、Excel。需要 agent 能接觸檔案,但不需要 git 操作。
- **後端工程師**:跑 codebase、寫測試、改 lint、開 PR。需要完整的 dev workflow agent。
- **DevOps / SRE / 量化交易**:需要 24/7 long-running、headless 排程、跑在 VPS 上不關機。

**用同一個 surface 服務以上四種人,結果不是齊一,而是全員都覺得不順手。** 工程師覺得 GUI 太慢,非工程師覺得 CLI 太硬,DevOps 覺得 desktop app 不能放遠端。

正確的做法不是統一 surface,而是**統一 engine + 統一 governance**,然後讓 surface 跟著角色走。

---

## 二、Anthropic 給你的是一個 surface 矩陣,不是一個產品

理解這件事是整個架構的基石。Anthropic 在 2025 年下半年到 2026 年初快速擴張 Claude Code 的 surface,現在實際上是這樣的拓撲:

```
                    ┌─────────────────────────┐
                    │   Claude Agent SDK      │  ← 共同引擎
                    │   (同一個 model loop)    │
                    └────────────┬────────────┘
                                 │
        ┌──────────┬─────────────┼──────────────┬────────────┐
        ▼          ▼             ▼              ▼            ▼
   ┌────────┐ ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌─────────┐
   │ Chat   │ │ Cowork   │ │ Desktop    │ │ Claude   │ │ Routines│
   │ (web/  │ │ (desktop │ │ Code tab   │ │ Code CLI │ │ (cloud  │
   │  app)  │ │  app tab)│ │ (desktop)  │ │ (term.)  │ │ schedul)│
   └────────┘ └──────────┘ └────────────┘ └──────────┘ └─────────┘
        │          │             │              │            │
        └──────────┴─────────────┴──────────────┴────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │  共用 config 層          │
                    │  CLAUDE.md / plugins /  │
                    │  MCP / hooks / skills   │
                    └─────────────────────────┘
```

**三件事是這個矩陣的核心:**

1. **共用 engine。** Cowork、Desktop Code tab、CLI 三者底層是同一個 Claude Agent SDK,Anthropic 的 docs 和獨立評測都明確指出這點。能力差異不在「模型聰不聰明」,而在 surface 包裝的工作流。
2. **共用 config。** CLAUDE.md、plugins、MCP servers、hooks、skills、permission settings 在 CLI 和 Desktop Code tab 之間是共用的。Plugin marketplace 標題寫的是 "Plugins for Claude Code and Cowork",代表治理層可以跨 surface 一致。
3. **不共用的是執行位置與互動模式**。CLI 跑本機;Desktop Code 跑本機 / 遠端 SSH / Anthropic cloud(三選一);Cowork 一定跑 Anthropic cloud VM;Routines 純雲端排程。這個差異決定了哪些架構模式做得到、哪些做不到。

---

## 三、Surface × 角色:正確的對應表

下面這張表是我推薦給公司的對應原則:

| 角色 | 主用 surface | 為什麼 | 何時跨界 |
|---|---|---|---|
| **PM / Operations / 行政** | Cowork tab | 零 terminal 門檻、檔案自動化、跨 app 工作流(Slack / Chrome connector / 本機檔案) | 需要看工程進度時,讀 `kanban.json` 即可 |
| **法務 / 財務 / 分析師** | Cowork tab | 合約整理、報表生成、SQL 後處理。Cowork 的 cloud VM 隔離得乾淨,不會污染本機 | 需要把資料丟給工程處理時,輸出進共用 repo |
| **前端 / 後端工程師(GUI 派)** | Desktop Code tab | sidebar 管多 session、視覺化 diff、live preview、PR 監控、跟 CLI 共享 config | 跑 long task / 需要 plugin debug 時切 CLI |
| **工程師(power user)** | Claude Code CLI | tmux 物理化 agent、headless `claude -p`、可自由腳本化、可放 VPS | 偶爾需要 GUI diff 時切 Desktop Code |
| **DevOps / SRE / Quant** | CLI + VPS | 24/7、cron、SSH、tmux detach。GUI surfaces 都做不到 long-running infrastructure | 需要快速 ad-hoc 調查時切 Desktop Code |
| **訪客 / 純諮詢 / 寫作** | Chat | 不需檔案存取的對話 | — |

**關鍵原則:不要把 Cowork 跟 Desktop Code 看成「初階版」與「進階版」的關係**。它們是**任務模式不同**:Cowork 是「給目標、走開、回來看結果」(autonomous task);Desktop Code 是「我跟 agent 一起在 codebase 裡互動」(interactive coding)。一個人在不同情境會切換 tab,完全合理。

---

## 四、1:1:1 物理化在這個矩陣裡的位置

Multi-agent 架構的第一塊基石是 **1:1:1 對應**:一個 agent = 一個 tmux pane + 一個 Claude session + 一個 GitHub repo(或 worktree)。這個物理化讓 agent 變成「看得到、改得到、復原得到」的 first-class 資源。

但這個模式**不是所有 surface 都能跑**:

| Surface | 1:1:1 物理化是否成立 | 怎麼做 |
|---|---|---|
| **CLI** | ✅ 完整 | tmux + git worktree + N 個 `claude` session,完整自由度 |
| **Desktop Code tab** | ✅ 部分 | 內建 sidebar 已經是「一個 agent 一個面板」的視覺化,等同於官方版 1:1:1。差別是 tmux 的物理化被 sidebar 取代,自由度低一點但不需要學 tmux |
| **Cowork tab** | ❌ | Cowork 一個 task 就是一個 cloud VM session,不是設計給多 agent 平鋪 |
| **Chat** | ❌ | 沒有 agent loop |

**這意味著:1:1:1 是給 CLI 和 Desktop Code 用戶的架構,給 Cowork 用戶不適用。** Cowork 用戶不需要 1:1:1,他們的 agent 是 task-based,一個任務一個 cloud VM,Anthropic 已經幫他們 sandbox 好。

---

## 五、VPS Tier 仍然是 CLI 專屬

對需要 24/7、long-running、定時排程的 agent —— 例如夜間跑 quant backtest、定時抓資料、監控 production system 的 ops agent —— 唯一可行的部署是:

```
VPS + tmux + claude -p (headless) + cron / webhook
```

**Desktop App 的所有 surface 都做不到這件事**,因為它們都需要 desktop app 開著。Anthropic 自家的 Routines 是 cloud 排程版本,但目前不適合需要完整 codebase 操作的 long-running infra agent。

這不是 surface 的缺陷,是設計取捨:GUI surface 換來的是非工程師也能用,代價就是綁定 desktop app 的 lifecycle。如果你需要的是「機器睡著它還在跑」,選 CLI + VPS,沒有第二個選項。

順帶一提:Desktop Code tab 有一個 **SSH mode**,可以連 VPS 上的 Claude Code session,本機關 lid 也不會中斷遠端 session。這對 GUI 派想用 VPS 的工程師是個有用的橋。

---

## 六、Governance 層:用 plugin 把所有 surface 黏合

公司有 PM 在用 Cowork、工程師在用 Desktop Code、SRE 在用 CLI + VPS —— 怎麼讓他們的工作能對接?

答案是**plugin governance**,而且這層在 Anthropic 的設計裡**是跨 surface 的**。

我推薦的 plugin 組合是 [`claude-workbench`](https://github.com/kirinchen/claude-workbench),它把以下四件事打包:

| Plugin | 作用 | 跨 surface 行為 |
|---|---|---|
| `kanban` | 用 `kanban.json` 當共用 task queue,人類和 agent 共寫共讀 | PM 在 Cowork 開 task,工程師 CLI agent 接走完成,所有人在同一張看板上 |
| `notify` | 透過 Pushover 推通知到手機 | 不論哪個 surface 卡在需要決策,都能找到人 |
| `mentor` | Epic / Sprint / Issue / ADR 文件層級規範 | 任何 surface 上的 agent 接 repo 都會被強制讀 bootstrap docs |
| `memory` | 跨 session RAG 記憶(本地 SQLite + embedding) | 跨 surface 的 agent 共享項目記憶 |

**關鍵價值:不論員工用什麼 surface,他們的 agent 行為都會被 plugin 收斂到同一套 SDLC 紀律。** 這是「異質 fleet」變成「同質行為」的唯一現實做法。

---

## 七、跨角色協作的具體場景

抽象講完,看三個真實場景就懂了:

**場景一:PM 開 task,工程師 agent 接走**
PM 在 Cowork tab 用自然語言描述「需要一個 endpoint 接收 webhook、寫進 DB、回傳 200」。Cowork 把這個 task 寫進 repo 的 `kanban.json`(透過 `kanban` plugin)。後端工程師早上開 CLI,跑 `/kanban:next`,Claude 自動挑了這個 TODO,在 tmux pane 裡開始實作。完成後自動 commit、轉 DONE,Pushover 通知 PM。**全程沒有人類手動 hand-off。**

**場景二:法務整理合約 → 工程師接著做合約分析系統**
法務在 Cowork tab 整理一批 NDA,輸出結構化 JSON 進共用 repo。工程師 CLI agent 看到 `kanban.json` 裡有「為這批資料寫 search API」的任務,自動接手。**法務不需要學 git,工程師不需要看原始合約。**

**場景三:SRE 的 long-running monitor agent**
SRE 在 VPS 上跑一個 24/7 監控 agent(CLI + tmux + cron)。當生產環境出現異常,agent 自動寫 issue 進 `kanban.json`,觸發 `notify` plugin 推到 on-call 手機。on-call 從手機開 Desktop Code tab,SSH 進該 VPS 接手處理。**緊急響應流程跨了三個 surface(CLI 偵測、Cowork-style 看板紀錄、Desktop Code 介入),但是同一份 governance 規則。**

---

## 八、落地步驟

不需要一次到位,建議四階段:

**Phase 1(1-2 週)— 全員裝 Claude Desktop App,選對的 tab。**
這是門檻最低的第一步:不需要學 terminal、不需要動 git。PM / 行政開 Cowork tab,工程師開 Code tab。光是這一步就能讓所有人從 Claude.ai 純 chat 升級到能執行任務的 agent。

**Phase 2(2-4 週)— 工程組進階導入 CLI 與 1:1:1。**
工程師學 tmux、git worktree、headless mode,開始用 CLI 跑 multi-agent。Desktop Code tab 仍可保留作為 GUI 補充,兩者共用 config。

**Phase 3(4-8 週)— 全員裝 `claude-workbench` plugin。**
這是治理層落地的關鍵。新進員工 onboarding 第一天裝 Desktop App、第二天裝 plugin marketplace,從此 task 都走 `kanban.json`、決策都走 Pushover、文件結構都走 mentor 規範。**跨 surface 的行為一致性從此被 plugin 強制執行。**

**Phase 4(2-3 個月)— Long-running agent 上 VPS,跨角色 kanban 上線。**
挑 1-2 個明確的 long-running use case 上 VPS(CLI 部署)。同時把 PM 在 Cowork 的 task 流跟工程的 CLI agent 流用 `kanban.json` 串起來。這時整個公司的 AI 工作流就 close loop 了。

---

## 九、結語

過去一年多 Anthropic 推出多個 Claude Code surface 的真正用意,不是要工程師選一個用,而是**讓不同角色都能進入 agentic 工作流**。Cowork 不是「給不會 code 的人的閹割版 Claude Code」,它是 PM、法務、分析師通往 agent 世界的入口;CLI 也不是「power user 的炫技工具」,它是 24/7 infrastructure agent 的唯一現實選項。

公司導入 multi-agent 架構的成功與否,不在於「全員用最強的工具」,而在於**讓每個角色用對自己的 surface,然後在 plugin governance 層讓他們的工作能對接**。三個 surface,一套 engine,一張看板 —— 這是 2026 真正可行的多人多 agent 架構。

異質的 fleet 不是問題,是 feature。