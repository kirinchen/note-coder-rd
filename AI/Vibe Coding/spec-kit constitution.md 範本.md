#### 1. 核心開發哲學 (Core Philosophy)
* **MVP 精神 (Minimum Viable Product)**：嚴格專注於 `/speckit.specify` 中定義的當下需求。只實作能滿足該業務邏輯的最少程式碼，優先交付具備核心價值的可運作版本。
* **拒絕過度設計 (No Overdesign)**：堅守 YAGNI (You Aren't Gonna Need It) 與 KISS 原則。嚴禁為了「未來可能會用到」的假想情境去預先建立複雜的介面或過深的抽象層。
* **高可維護性與可擴充性 (Maintainable & Extensible)**：系統設計必須符合開閉原則 (OCP)。確保未來新增功能時，只需「擴充」新模組，而不需要大幅修改或破壞既有的程式碼，讓專案能長期健康迭代。

#### 2. 架構與程式碼品質 (Architecture & Code Quality)
* **拒絕重複代碼 (DRY over WET)**：嚴格遵守 DRY (Don't Repeat Yourself) 原則，絕對避免 WET (Write Everything Twice)。必須將共用的邏輯抽取為獨立的模組、共用函式或常數，消除所有不必要的重複程式碼。
* **拆分檔案與拒絕上帝物件 (No God Object/Function)**：堅守單一職責原則 (SRP)。檔案、類別與函式必須盡量細粒度拆開。**嚴禁產生包山包海的 Super Class (超級類別) 或動輒數百行的 God Function (上帝函式)**。
* **職責分離 (Separation of Concerns)**：維持高內聚、低耦合的設計。商業邏輯必須與框架細節（如 HTTP 請求處理、資料庫連線）完全隔離。
* **官方命名規範 (Naming Conventions)**：嚴格遵守該語言與框架的官方命名慣例。特別注意跨系統/跨語言的資料邊界（如 REST API 的 JSON 統一為 `camelCase`，資料庫映射為 `snake_case`）。
* **防禦性設計與錯誤處理**：避免靜默失敗 (Silent Failures)。在系統邊界（API 輸入、資料庫讀寫、外部服務呼叫）必須實作清晰的錯誤捕捉與合適的 HTTP 狀態碼回傳。

#### 3. 測試規範 (Testability - Testable By Design)
* **可測試性優先**：所有核心商業邏輯都必須是「易於單元測試」的。依賴項（如外部 API 或資料庫）應具備可被 Mock（模擬）的設計。
* **有意義的自動化測試**：每一項交付的功能模組，必須附帶對應的自動化測試（單元測試或整合測試），專注於驗證商業邏輯與邊界條件。

#### 4. AI 協作準則 (AI Agent Guidelines)
* **不擅自發明需求**：若 `/speckit.specify` 的需求描述有模糊地帶，AI 必須主動提問要求澄清，嚴禁自行腦補並實作未經確認的商業邏輯。
* **嚴守技術棧限制**：在生成程式碼時，必須絕對服從 `/speckit.plan` 中所指定的語言、框架與套件版本，嚴禁擅自引入未經授權的第三方依賴或工具。
