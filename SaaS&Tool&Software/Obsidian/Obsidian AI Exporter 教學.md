
# Obsidian AI Exporter 完整教學：把網頁 AI 對話變成 Markdown

你很可能遇過這種情況：在 **Gemini**、**ChatGPT**、**Claude** 或 **Perplexity** 裡聊了很久，結論、程式碼、大綱都留在瀏覽器分頁裡；過幾天要找回「當時那一句」，只能在搜尋框、歷史紀錄裡翻來翻去。**Obsidian AI Exporter** 這類 Chrome 擴充功能的目的很單純：在支援的 AI 網頁上按一下，把整段對話整理成**結構化 Markdown**，讓你可以貼進 Obsidian、Notion、VS Code，或任何吃 Markdown 的地方。

本篇依**由淺入深**順序撰寫：**先**用**複製剪貼簿**（不必裝 Obsidian 外掛）；需要**一鍵寫進 Obsidian 庫**時，再設定 **Local REST API**。

---

## 這套工具在做什麼？

**Obsidian AI Exporter**（開發者 [sho7650](https://github.com/sho7650/obsidian-AI-exporter)，MIT 授權）是跑在 **Google Chrome**（桌面版）上的擴充功能。它會在支援的 AI 網頁上顯示一顆常見為**紫色**的 **Sync** 按鈕，把目前對話匯出為：

- **YAML frontmatter**（標題、來源、網址、時間、標籤等，依設定與版號而異）
- **Obsidian Callout** 語法的問答區塊（例如 `[!QUESTION]`、`[!NOTE]`）
- 盡量保留程式碼區塊、清單、連結等 Markdown 結構

官方標榜處理在**本機瀏覽器**完成、**不把對話送到第三方伺服器**做轉檔（實際行為請以 [GitHub README](https://github.com/sho7650/obsidian-AI-exporter) 與商店說明為準）。

---

## 適合誰？

- 用 **Obsidian** 做筆記，但對話散落在各 AI 網站的人  
- 想保留 **原始對話網址**、方便日後查證與搜尋的人  
- 希望同一則內容可以**貼到多種工具**（文件、PR、Notion 等）的人  
- 進階：希望 **Cursor**、**Claude Code**、**Gemini CLI** 等工具能讀到**同一套本機 `.md` 檔**的人  

---

## 使用前的限制（先看這段）

| 項目 | 說明 |
| --- | --- |
| **瀏覽器** | 需 **Chrome 桌面版**（或相容的 Chromium）。手機版 Chrome 無法安裝同一套流程。 |
| **網址** | 需在擴充功能支援的網域開啟對話（見下方「支援平台」）。 |
| **Obsidian** | **只有「直存進庫」**需要 Obsidian 安裝 **Local REST API**。**僅剪貼簿／下載**時，Obsidian 只當「貼上的編輯器」即可，不必先裝該外掛。 |
| **版號** | UI 文案、按鈕位置、支援範圍可能隨更新改變，請以 **Chrome 商店頁**與 **GitHub Release** 為準。 |

---

## 支援平台（網域）

依專案 README 慣例，通常包含（實際以你安裝的版本為準）：

- **Google Gemini**：`gemini.google.com`  
- **Claude**：`claude.ai`  
- **ChatGPT**：`chatgpt.com`  
- **Perplexity**：`www.perplexity.ai`  

部分版本另支援 **Deep Research**、**Claude Artifacts**、**Append（只追加新訊息）** 等，請在 GitHub 看當前說明。

---

## 第一步：安裝 Obsidian AI Exporter

1. 開啟 Chrome，前往 **Chrome 線上應用程式商店**：  
   [Obsidian AI Exporter](https://chromewebstore.google.com/detail/obsidian-ai-exporter/edemgeigfbodiehkjhjflleipabgbdeh)  
2. 點 **加到 Chrome** → 依提示完成安裝。  
3. 必要時在工具列**釘選**擴充功能圖示，方便之後點 **Popup** 設定。

安裝後，到任一**支援的 AI 網站**開啟一則對話，畫面上應會出現 **Sync**（位置依 UI 改版可能不同，多在右下角附近）。

---

## 方式一：複製到剪貼簿（最推薦入門）

**不需要**先設定 Local REST API，也不需要先在 Obsidian 裝任何東西（只要你能開一個空白筆記貼上即可）。

1. 在已登入的 AI 網頁開啟要匯出的對話（建議用**可公開示範**的內容，勿錄敏感資料）。  
2. 必要時**捲動**讓長對話載入完整（部分站台支援自動捲動載入，依版號為準）。  
3. 點 **Sync**。  
4. 在選單中選擇類似 **Copy to clipboard**／**複製剪貼簿** 的選項（實際文案以畫面為準）。  
5. 開啟 **Obsidian** 新建筆記，或開啟 VS Code、Notion、記事本等，**貼上**（Ctrl+V / Cmd+V）。  

你應會看到帶 **frontmatter** 的 Markdown，以及分段清楚的 **Callout** 或標題，並通常含**來源網址**，方便之後點回去原對話。

**優點**：步驟最少、觀眾與讀者最容易跟著做。  
**缺點**：每次要自己貼上、自己存檔；若要固定路徑與檔名規則，適合改用方式三。

---

## 方式二：下載為 `.md` 檔

流程與方式一類似，在 **Sync** 後選擇**下載檔案**（若有此選項），將 `.md` 存到本機後：

- 把檔案**拖進 Obsidian 的 vault 資料夾**，或  
- 用檔案總管複製到 Git 庫、專案目錄  

適合想**先存檔再決定放哪個資料夾**的人。

---

## 方式三：一鍵寫入 Obsidian（Local REST API）

適合已固定用 Obsidian、希望 **Sync 後直接出現在庫裡**、並可搭配 **Vault 路徑模板**（例如依平台分資料夾）的使用者。

### 3.1 在 Obsidian 安裝 Local REST API

1. 開啟 Obsidian → **設定** → **社群外掛** → 關閉安全模式（若尚未關閉）。  
2. **瀏覽** → 搜尋 **Local REST API**（專案見 [GitHub：obsidian-local-rest-api](https://github.com/coddingtonbear/obsidian-local-rest-api)）。  
3. **安裝**並**啟用**。  
4. 進入該外掛設定：記下 **API URL**（常見預設為 `http://127.0.0.1:27123`，實際以你畫面為準）。  
5. **產生或複製 API Key**，並**妥善保管**（不要貼在公開影片或截圖裡）。

### 3.2 在 Chrome 擴充功能 Popup 填寫

1. 點工具列上的 **Obsidian AI Exporter** 圖示開啟 Popup。  
2. 填入 **API Key**、**API URL**（與 Obsidian 外掛一致）。  
3. **Vault Path**：填相對於 vault 根目錄的儲存路徑，例如 `AI/{platform}`（`{platform}` 這類變數是否支援、語法為何，請查當版 [GitHub README](https://github.com/sho7650/obsidian-AI-exporter)）。  

### 3.3 匯出

回到 AI 對話頁 → **Sync** → 選擇寫入 **Obsidian**（或介面上對應「直存庫內」的選項）。成功後在 Obsidian 中應出現新檔或更新既有筆記（若使用 Append 模式，依設定而異）。

**注意**：需保持 **Obsidian 正在執行**，且 Local REST API ** listening** 在你填的埠上；防火牆或 HTTPS 進階設定請參考 Local REST API 文件。

---

## 匯出後的筆記長什麼樣子？

常見包含：

- **Frontmatter**：`title`、`source`、`url`、`tags`、`date` 等（欄位名稱以實際輸出為準）  
- **Callout**：區分使用者提問與 AI 回答，並在 Callout 類型或標題標示平台名稱（依擴充功能設定）  
- **程式碼**：以 fenced code block 保留語言標籤（若網頁端有標記）  
- **LaTeX**：部分版本標榜保留 `$...$` 或 `$$...$$`（仍以實測為準）  

匯入後你可以在 Obsidian 裡加 **wikilink**、**標籤**、**Dataview**，或交給 **Claude Code** 在庫內做檔名與資料夾整理。

---

## 隱私與安全建議

- **API Key** 等同進入你 vault 的鑰匙，勿提交到 Git、勿完整入鏡教學影片。  
- 示範對話請用**無敏感個資、無機密程式碼**的內容。  
- 口播與文章請避免讓人誤以為 **Google／OpenAI／Anthropic** 等**官方背書**此擴充功能。  

---

## 常見問題（簡答）

**Q：按了 Sync 沒反應？**  
檢查是否在支援網域、頁面是否載入完成、擴充功能是否啟用；可試重新整理頁面或更新擴充功能版本。

**Q：直存 Obsidian 失敗？**  
確認 Obsidian 已開、Local REST API 已啟用、URL 埠與 Key 與 Popup 完全一致；本機防火牆是否擋連線。

**Q：和「只截圖」有什麼差？**  
Markdown 可搜尋、可版本控制、可再編輯與連結到其他筆記，適合長期知識庫。

**Q：一定要用 Obsidian 嗎？**  
不必。剪貼簿與下載檔可進任何工具；**只有一鍵進庫**綁 Obsidian + Local REST API。

---

## 延伸：為什麼很多人會接到 Obsidian？

簡短類比：**Notion** 等常把內容放在**雲端工作區**與帳號權限下；**Obsidian** 預設是**本機資料夾裡的一堆 `.md`**，較容易與 **Git、Cursor、終端機裡的 CLI** 指到同一路徑。你若關心 **Local first**、離線可讀、備份自管，Obsidian 路線會很順；若你只需要偶爾備份對話，**剪貼簿貼到單一筆記**就夠用。

---

## 官方連結整理

| 資源 | 網址 |
| --- | --- |
| Chrome Web Store | https://chromewebstore.google.com/detail/obsidian-ai-exporter/edemgeigfbodiehkjhjflleipabgbdeh |
| GitHub（原始碼與 README） | https://github.com/sho7650/obsidian-AI-exporter |
| Obsidian Local REST API | https://github.com/coddingtonbear/obsidian-local-rest-api |

開發者亦曾在 [DEV.to](https://dev.to/sho7650/obsidian-ai-exporter-the-current-evolution-supporting-multiple-ais-32kn) 等處分享功能演進，可作補充閱讀。

---

## 免責與版本聲明

本文描述之步驟與 UI 名稱以撰寫當下之公開文件為參考；**擴充功能與各 AI 網站介面可能隨時更新**。實作前請以 **Chrome 商店**、**GitHub 最新 README** 為準。本文與 Obsidian AI Exporter 開發者無僱傭或代言關係，僅為教學整理。

---

*本教學與同資料夾內 [[影片企劃/Obsidian AI Exporter 讓你的AI Chat Data移至你的 Obsidian/企劃概要|企劃概要]]、[[影片企劃/Obsidian AI Exporter 讓你的AI Chat Data移至你的 Obsidian/腳本與分鏡|腳本與分鏡]] 之影片企劃對齊；修訂時建議一併更新。*