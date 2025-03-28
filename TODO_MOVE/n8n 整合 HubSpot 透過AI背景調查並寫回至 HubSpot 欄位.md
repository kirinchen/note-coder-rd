---
title: n8n 整合 HubSpot 透過AI背景調查並寫回至 HubSpot 欄位

---

# n8n 整合 HubSpot 透過AI背景調查並寫回至 HubSpot 欄位

這裡是一份俏皮風格的教學文件，介紹如何使用 n8n 整合 HubSpot 進行 AI 背景調查，並將結果寫回 HubSpot 欄位。

---

# 🚀 用 n8n 讓 HubSpot 更聰明！  
## *為什麼要用 n8n？傳統方法 VS 自動化的革命性差異！*

**傳統方法：**  
🙄 手動輸入公司資訊，然後...  
🔍 花大把時間 Google 搜尋公司背景，然後...  
📝 手動整理資訊，然後...  
📤 再慢吞吞地回填 HubSpot。

這是 2025 年了，我們不該還在用石器時代的方式處理數據！  

**n8n 自動化流程：**  
⚡ 當有新公司被建立時，n8n 自動啟動！  
🤖 AI 自動爬網、分析，並給出評分！  
📌 直接將背景調查結果寫入 HubSpot，超快速！  

現在，來看看這個超酷的 n8n workflow 是怎麼運作的吧！🚀

---

## 🔧 n8n + HubSpot AI 背景調查工作流詳解  

### **Step 1: 觸發事件 - 新公司建立**
🎯 當 HubSpot 有新的公司記錄建立時，**HubSpot Trigger** 節點 (HubSpot Trigger) 自動偵測，讓整個流程開始運作。

---

### **Step 2: 取得公司資訊**
📡 **HubSpot 節點 (HubSpot Get Company Data)** 透過 API 取得該公司的**商業登記號碼** (Business_Accounting_NO)。

---

### **Step 3: 查詢政府公開數據**
🔎 **HTTP Request 節點 (OrigReg1 & OrigReg3)**  
- 我們直接向 **台灣經濟部** 的開放資料 API 查詢這家公司的基本資訊，如：  
  ✅ 公司名稱  
  ✅ 註冊地址  
  ✅ 成立時間  
  ✅ 負責人  

---

### **Step 4: AI 背景調查**
🧠 **AI 節點 (Generate Summary AI)**  
這裡是魔法發生的地方！✨  
我們用 GPT-4o-mini 幫你做以下事情：
1. 🔎 **Google 搜尋** 該公司，確認是否有負面新聞或其他資訊。  
2. 📖 **Wikipedia 查詢**（如果公司夠大！）  
3. 📝 **總結資訊並評分**（1-10 分，代表該公司可靠程度）。  

輸出的 JSON 格式：
```json
{
  "content": "### 公司基本資訊\n(詳細內容...)\n\n### Google 搜尋評估\n(詳細內容...)\n\n### 總體評估\n(總結結論)",
  "score": 8.5
}
```

---

### **Step 5: 更新 HubSpot**
💾 **HubSpot Update 節點 (HubSpot1)**  
- 將 AI 產出的分析結果填入 **HubSpot「關於我們」(About Us) 欄位**  
- 讓你的銷售或商業開發團隊，隨時都能看到這家公司的背景！

---

### **Step 6: 格式轉換 & Email 通知**
📩 **Markdown 轉換 & Gmail 節點 (Markdown & Gmail)**  
- 轉換 AI 分析結果為 HTML 格式  
- 自動寄送 email 給內部團隊，例如 `service@domiearth.com`，讓大家第一時間掌握資訊！🔥

---

## 🎉 這樣，你的 HubSpot 瞬間升級成 AI 商情神器！  

現在，你不再需要：
❌ 手動 Google 搜尋  
❌ 一個個填寫公司背景  
❌ 擔心遺漏重要訊息  

n8n 會幫你做完所有事情，讓你專注於更重要的決策！🚀

---

這樣的工作流是不是超酷？快來試試看，把你的 HubSpot 玩出新高度吧！😎