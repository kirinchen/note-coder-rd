# 🚀 企業級 Spec-Driven Development (SDD) 開發指南與範本庫

歡迎來到公司的 AI 開發範本庫！本專案旨在透過 **Spec-Driven Development (規格驅動開發)** 流程，標準化 AI Agent (如 Cursor, Claude Code) 的開發產出。

透過預先定義好的「專案憲法」與「技術實作藍圖」，我們能有效防止 AI 寫出難以維護的義大利麵條程式碼、包山包海的 God Function，並確保所有產出符合 **MVP (最少可行性產品)** 與 **可測試性 (Testable)** 的核心精神。

---


---

## 🛠️ 1. 環境安裝 (Prerequisites)

在開始之前，請確保你的開發機已安裝 GitHub 官方的 `spec-kit` CLI 工具。

### 步驟 A：安裝 `uv` (極速 Python 環境管理器)
請打開 **PowerShell** 執行以下指令進行獨立安裝（強烈建議不要用 `pip` 裝，以確保環境隔離）：
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

```

_(安裝完成後，請務必重啟終端機)_

### 步驟 B：安裝 `specify-cli`

在終端機執行以下指令，透過 Git 拉取並全域安裝：



```Bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

```

輸入 `specify check`，若正確列出環境狀態即代表安裝成功！

----------

## 🏃‍♂️ 2. 標準使用流程 (Workflow)

當你要啟動一個新專案或新功能時，請在你的 AI 編輯器（如 Cursor）中依照以下步驟執行：

### Step 1: 初始化專案

建立並打開一個空資料夾，在終端機輸入（以下以 Cursor 為例）：



```Bash
specify init . --ai cursor-agent

```

這會在你的目錄下建立 `.speckit` 資料夾。

### Step 2: 載入專案大憲章 (Constitution)

將本 Repo 提供的 `constitution.md` 內容，複製並覆蓋到專案中的 `.speckit/constitution.md`。

### Step 3: 喚醒 AI Agent 開始對話

打開編輯器的 AI Composer / Agent 模式，依序輸入以下指令：

1.  **定義需求**：
    
    輸入 `/speckit.specify` 並接上你這次要做的**商業需求**。
    
    _(例如：`/speckit.specify 幫我做一個可以上傳 CSV 並過濾空值的 ETL 腳本...`)_
    
2.  **套用技術藍圖**：
    
    輸入 `/speckit.plan` 並貼上本 Repo 中**對應的技術範本內容**（如 `plan-template-python-etl.md`）。
    
3.  **確認任務拆解**：
    
    輸入 `/speckit.tasks`，檢查 AI 列出的實作步驟是否符合模組化、無 God Function 的原則。
    
4.  **開始實作**：
    
    確認無誤後，輸入 `/speckit.implement`，讓 AI 開始撰寫具備測試的高品質程式碼！
    

----------

## ❓ 3. 常見問題 (Q&A)

### Q1: 如果這是「已經進行中的專案（舊專案）」，該如何使用？

**A:** `spec-kit` 非常適合用在既有專案（Brownfield Development）中擴充新功能！

1.  **初始化**：直接在你現有的專案根目錄執行 `specify init . --force --ai cursor-agent`（使用 `--force` 略過空目錄檢查）。
    
2.  **限制範圍**：在輸入 `/speckit.specify` 時，**清楚表明這是一個既有專案**。例如：`/speckit.specify 在現有的 Spring Boot 專案中，新增一支處理退款邏輯的 API。請分析既有的架構與命名慣例並嚴格遵循。`
    
3.  **使用專屬指令**：在產生 Plan 之前，建議先使用 `/speckit.clarify` 讓 AI 主動發問，釐清它對你現有程式碼架構的理解盲點，確認它不會破壞既有邏輯後，再進入 Tasks 與 Implement 階段。
    

### Q2: 為什麼要這麼麻煩分這幾個步驟，不能直接叫 AI 寫嗎（Vibe Coding）？

**A:** 直接叫 AI 寫（Vibe Coding）在寫單一腳本時很快，但當專案變大，AI 非常容易「失憶」，導致邏輯衝突、無視架構分層，最後寫出一坨無法測試的程式碼。透過 SDD 流程，我們強迫 AI 在寫 Code 前先確認「規則 (Constitution)」與「計畫 (Plan)」，這能省下未來數十倍的 Debug 與重構時間。

### Q3: AI 產生的 `/speckit.tasks` 任務清單我不滿意，可以直接改嗎？

**A:** **絕對可以，而且強烈建議這麼做！** 如果你發現 AI 試圖把所有邏輯塞進同一個檔案，或者忘記加入單元測試的任務，請直接在對話框糾正它（例如：「任務 3 太龐大了，請依照大憲章的單一職責原則，把它拆成三個獨立的 Utility Functions 並補上測試任務」），等它修正計畫後，再執行 `/speckit.implement`。

