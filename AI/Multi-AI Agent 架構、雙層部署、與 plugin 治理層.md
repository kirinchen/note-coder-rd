# Multi-AI Agent 架構:1:1:1 對應、雙層部署、與 plugin 治理層

> 寫給:還停在「一人一台 PC 跑一個 Claude」的團隊。
> 
> TL;DR — 把 agent 想成「一個 tmux pane + 一個 Claude Code session + 一個 GitHub repo」的三元組;短期 agent 留在開發機上,長期 agent 移到 VPS;然後用 plugin 把行為規範統一強制到所有機器上。這不是個人偏好,是 2026 業界正在收斂的形狀。

---

## 一、問題:單機單 Claude 的天花板

公司目前的形態是這樣的:每個工程師在自己的 PC 上開一個 Claude(或 Claude Code)session,要做事就打開、用完就關掉。短期內看起來很合理,因為 IDE-like 的互動體驗很順,而且不需要任何架構成本。

但只要你開始用 Claude Code 做點真正有重量的事,這套會立刻撞牆:

- **單一 session 就是單一 context 瓶頸。** 你想同時讓 Claude 重構 backend、改 frontend、寫 test,只能讓它一件一件做,因為一個 session 一個 context 視窗。
- **關 lid 就斷氣。** 想跑 overnight backtest、定時抓資料、長期觀察某個系統 —— 只要關筆電就停。這對 quant 跑參數最佳化、或長期運行的 ops agent 是致命的。
- **跨機器不一致。** A 同事的 Claude 會自動跑 lint、會 commit kanban 狀態;B 同事的不會。沒有統一規範,output 品質和工作流會嚴重 drift。
- **沒有 task queue,沒有 audit trail。** 你問 Claude 上週幫你做了什麼,它不知道。人類也不知道。

要解決這四個痛點,單純買更貴的訂閱沒用。需要的是**架構**。

---

## 二、設計核心:1:1:1 對應原則

整套架構的第一個基石是這個對應:

```
┌──────────────┐     ┌────────────────────┐     ┌──────────────┐
│  tmux pane   │ ←→  │ Claude Code session│ ←→  │  GitHub repo │
└──────────────┘     └────────────────────┘     └──────────────┘
   物理載體              邏輯 agent              事實來源/工件
```

**一個 agent = 一個 tmux pane + 一個 Claude Code session + 一個 GitHub repo**,嚴格 1:1:1。為什麼?

**Context 隔離。** 每個 Claude Code session 自帶一個 context 視窗,把它鎖在自己的 repo 裡,不會被其他 agent 的工作打斷或污染。Anthropic 的 Agent Teams 設計就是這個原則 —— 每個 teammate 自己一個 context,team lead 只透過 message 協調,不共用 context。

**可觀察性。** tmux 把每個 pane 鋪在同一個畫面上,你一眼可以看到 5 個 agent 同時在做什麼。pane 就是 agent 的物理化呈現。EPAM 的 Octobots、smtg-ai 的 claude-squad,以及一票社群方案,全部都是這個模式 —— 因為「能看到 agent 在做什麼」對信任和除錯都是不可妥協的。

**可恢復性。** SSH 斷線、筆電休眠、網路抖動 —— tmux session 在,Claude Code session 就還在。這對 VPS 場景特別關鍵,但其實在本機也省下無數次「啊我剛剛那個 context 沒了」的痛苦。

**Repo 即事實來源。** 每個 agent 對應一個 repo(或一個 worktree)是個非常乾淨的設計:agent 的 spec 寫在 `CLAUDE.md` / `specs/*.md`,工作成果是 commit,狀態是 `kanban.json`。沒有任何東西藏在記憶體裡。換機器、換人接手、跑 review,都只要看 repo。

> **一個值得記住的反面教材:** 如果你讓兩個 Claude session 跑在同一個 repo 上、共用 working tree,它們會互相覆蓋彼此的 edit。這不是 Claude 的 bug,這是把 1:1:1 打破變成 2:1 的後果。要平行就用 git worktree,讓每個 agent 有自己的 working dir。

---

## 三、雙層部署:本機開發機 vs VPS

光有 1:1:1 還不夠。你還需要決定每個 agent **跑在哪台機器上**。

### Tier 1 — 本機開發機(高規格 Linux/macOS workstation)

**用途:** 設計階段、prototyping、需要密集人機互動的短期 agent。

**典型工作流:**

- 早上開 tmux,split 出 3 個 pane,分別跑 backend agent / frontend agent / test agent。
- 寫完 spec 丟給對應的 Claude Code session,讓它們各自工作,你監看。
- 收工時 commit、push、關機。

**為什麼留在本機:** 這類 agent 需要你頻繁介入(看 diff、批准、改 prompt),延遲敏感。而且開發機通常 RAM 大、SSD 快、有多核可以同時跑 build/test —— Anthropic 自己的 Agent SDK hosting 文件也提醒,multi-agent 同時跑會吃很多記憶體。

### Tier 2 — VPS(長期、headless、不能關機的)

**用途:** 24/7 觀察、定時排程、不需要人盯的 long-running agent。

**典型工作流:**

- VPS 上開 tmux session,跑 `claude -p` headless mode,搭配 cron 或 webhook 觸發。
- agent 每小時去檢查資料源、跑模型推論、抓交易訊號、push 通知到 Pushover。
- 你出差、睡覺、放假,它都在跑。

**為什麼搬到 VPS:**

- **24/7 不中斷。** 筆電要關、要帶出門、要重開機;VPS 不會。
- **資源不互搶。** Quant backtest 跑兩天會把筆電風扇逼到尖叫;VPS 沒這個問題。
- **位置中立。** 從手機、平板、別人家電腦 SSH 進去都能繼續用同一個 session。

業界已有現成的 VPS 服務商把這個 use case 包成商品(QuantVPS、Hostodo、ClaudeFast 都是),代表這個分工已經被市場驗證。

### 為什麼是「兩層」而不是「全部上 VPS」或「全部留本機」

這是一個成本與互動性的取捨:

|維度|本機|VPS|
|---|---|---|
|互動延遲|極低|SSH 來回有 latency|
|資源彈性|看你買的機器|隨時升級|
|成本|已沉沒|按月付|
|可用性|看你關不關機|24/7|
|適合|互動式設計、實驗|排程、監控、長期任務|

短期 agent 上 VPS 是浪費(SSH 延遲拖慢 iteration);長期 agent 留本機是脆弱(關機就死)。所以**雙層**。

---

## 四、Governance 層:用 plugin 統一所有 agent 的行為

到這裡我們解決了「怎麼跑」和「跑在哪」,還剩最後一個問題:**怎麼讓 N 個 agent 表現得像同一個團隊?**

如果每個工程師的 Claude Code 都是裸裝、各自定義工作流,你會得到 N 種風格的 commit message、N 種 task 命名規則、N 種「我跑完了」的判定方式。這跟一個沒有 coding standard 的團隊是同一個問題,只是把人換成 AI。

解法是 **plugin 治理層**。Anthropic 從 2025 年下半年起就把 plugin marketplace 做進 Claude Code,就是要讓團隊可以把 SDLC 規範打包,在所有機器上一鍵安裝、強制套用。

我推薦的 plugin 組合是 [`claude-workbench`](https://github.com/kirinchen/claude-workbench)(我自己維護的),它是一組互相補足的 plugin:

|Plugin|解決什麼|
|---|---|
|`kanban`|用 `kanban.json` 當共用 task queue,人類和 agent 共寫共讀;agent 不能亂跳 dependency,不能改 DONE 欄,所有狀態轉移自動 commit。|
|`notify`|當 agent 卡在需要決策的點時,Pushover 推到你手機。不再需要每 15 分鐘回頭看 tmux。|
|`mentor`|規範 Epic / Sprint / Issue / ADR 的文件層級 —— 任何 agent 接手新 repo 都會被強制讀 bootstrap docs,不會「自己發明工作流」。|
|`memory`|跨 session 的 RAG 記憶(SQLite + embedding,純本地)。Agent 結束後學到的東西不會蒸發。|

**這層的價值不是炫技,是治理。** 想像公司有 10 個工程師、每人 3 個 agent,共 30 個 Claude session 同時在動。沒有 plugin governance,你就是 30 種隨機行為的混合;有 plugin governance,你拿到的是 30 個遵守同一份 SDLC 紀律的工人。

---

## 五、業界對齊:這個架構不是孤軍假設

如果這篇文章的提案聽起來「太前沿」,值得看看它和現有方案的對應關係:

- **Anthropic 官方 Agent Teams**(experimental):用 tmux 物理化 agent、team config 直接記 tmux pane ID —— 我們的 1:1:1 就是這個。
- **smtg-ai/claude-squad**:被廣泛認為是「跑多個 Claude Code agent」最成熟的社群方案,核心是 tmux + git worktree。
- **EPAM 的 Octobots**:6 個 Claude Code session 平鋪 tmux,跑 PM/BA/TL/Backend/Frontend/QA 六種角色,SQLite 當 message queue。完整重現一個軟體團隊。
- **Composio Agent Orchestrator**:平行 coding agent + worktree + CI 修復 + code review。
- **Anthropic 的 16-agent C compiler 案例**:16 個 Claude 平行跑了 ~2,000 個 session,合作做出能 boot Linux 6.9 的 compiler,成本不到 $20K。證明這套架構可以 scale 到很大。
- **Hermes Agent + Bluehost VPS**:整個產品定位就是「long-running agent 放 VPS」,跟我們 Tier 2 的設計同源。

換句話說,我們提出的不是個人實驗,是把 2026 上半年業界已經跑出來的 best practice **組合成一套適合公司導入的版本**。

---

## 六、給公司的落地步驟

不需要一次到位。建議分四階段:

**Phase 1(1-2 週)— 從 1 人 1 Claude 升到 1 人 N agent。** 全員裝 tmux、學會 split pane、學會 git worktree。在自己機器上開始用 1:1:1 模式跑 2-3 個 agent。這階段不用碰 VPS、不用碰 plugin,純粹是工作流升級。產出:每人都能同時讓 Claude 跑 backend + frontend + test。

**Phase 2(2-4 週)— 試點長期 agent 上 VPS。** 挑 1-2 個明確的 long-running use case(例如:每日資料抓取、夜間 lint 自動修、定時 PR review),在一台便宜 VPS 上跑起來。用 `claude -p` headless + tmux + cron。產出:第一個不需要任何人開機的 agent。

**Phase 3(4-8 週)— 全員導入 plugin governance。** 把 `claude-workbench` 加到公司的 onboarding。新進工程師第一天裝 Claude Code、第二天裝 workbench plugin,從此 task 都走 `kanban.json`、決策都走 Pushover、文件都走 mentor 規範。產出:跨機器、跨人員、跨 agent 的行為一致性。

**Phase 4(持續)— 人機共用 task queue。** 這是最有趣的一步:`kanban.json` 變成人類和 agent 共寫的看板。PM 在 Linear/Jira 開的票,自動 sync 進 kanban,某些標籤的票直接派給 agent;agent 做完自己 commit、自己關票。人類只在 BLOCKED 或 review 時介入。

---

## 七、結語

「一人一台 PC 跑一個 Claude」對嘗鮮夠用,對生產不夠用。

接下來這層需要的是工程紀律 —— 把 agent 當成一種有 lifecycle、有部署位置、有行為規範的 first-class 資源來治理。1:1:1 是物理結構,雙層部署是空間策略,plugin governance 是行為標準。三件事缺一不可。
