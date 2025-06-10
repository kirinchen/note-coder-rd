

# 用 n8n 打造無碼 Google 登入驗證流程 —— 前端 OAuth，後端驗證完全自動化

在現代網頁專案中，許多人選擇 Google 登入作為快速且安全的身分驗證解決方案。  
你知道不用自己寫 Node.js/Express 伺服器，只靠 n8n 這種無碼自動化工具，也能做嚴謹的 Google Login 驗證嗎？  
本篇就帶你用**n8n**快速實現：「前端取得 Google ID Token，n8n 完成安全驗證、後續自動化，零後端程式碼！」

---

## 一、架構流程圖

```text
[前端 React/JS]
    |
    | POST /google-login  (body: { id_token })
    ↓
[n8n Webhook Workflow]
    |
    |── HTTP Request: 驗證 id_token (呼叫 Google tokeninfo)
    |
    |── IF: 檢查 aud、iss、exp 等 claims
    |      ├─ True (合法): 進行業務邏輯、自動化
    |      └─ False: 回傳 401/錯誤訊息
    ↓
[資料庫、通知、回應前端...]
```

---

## 二、前端：取得 Google ID Token，傳給 n8n

假設你用 React，流程如下：

1. **引入 Google Identity Services：**
    
    ```js
    import { GoogleLogin } from '@react-oauth/google';
    ```
    
2. **取得 credential 並送給 n8n：**
    
    ```jsx
    <GoogleLogin
      onSuccess={credentialResponse => {
        fetch('https://your-n8n-domain.com/webhook/google-login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ id_token: credentialResponse.credential }),
        })
          .then(res => res.json())
          .then(data => {
            if (data.success) {
              // 登入成功，存使用者資料、跳轉頁面等
            } else {
              // 處理登入失敗
            }
          });
      }}
      onError={() => alert('Google 登入失敗')}
    />
    ```
    
3. - `id_token` 就是 Google 回傳給前端的 JWT，接下來就交給 n8n 處理！
        

---

## 三、n8n：無碼驗證 Google ID Token

**Step 1. Webhook 節點**

- 新增 Webhook 節點，Method 選 POST，Path 設定 `/google-login`。
    
- 預期前端會送進來 `{ id_token: <token> }`。
    

**Step 2. HTTP Request 節點（驗證 Token）**

- 新增 HTTP Request 節點，連到 Webhook 節點。
    
- 設定：
    
    - Method: `GET`
        
    - URL: `https://oauth2.googleapis.com/tokeninfo?id_token={{ $json["id_token"] }}`
        
- 此節點會呼叫 Google 的 TokenInfo API，驗證 token 是否有效。
    

**Step 3. IF 節點（檢查 Claims）**

- 新增 IF 節點，連接 HTTP Request 節點。
    
- 設定多個條件，全部都要成立（AND）：
    
    1. `{{$node["HTTP Request"].json["aud"]}}` **等於** 你自己申請的 Google Client ID
        
    2. `{{$node["HTTP Request"].json["iss"]}}` **等於** `accounts.google.com`
        
    3. `{{$node["HTTP Request"].json["exp"]}}` **大於** `{{ Math.floor(Date.now()/1000) }}`
        
    4. （可選）`{{$node["HTTP Request"].json["email_verified"]}}` **等於** `true`
        

**Step 4. Respond to Webhook 節點**

- 合法分支（True）：
    
    - 可以把驗證後的 Google 資料（email, name, sub...）回傳給前端，或做進一步自動化（如寫 DB、觸發郵件等）。
        
    - 範例回傳：
        
        ```json
        {
          "success": true,
          "email": "user@gmail.com",
          "name": "某某",
          "sub": "1076915035..."
        }
        ```
        
- 不合法分支（False）：
    
    - 回傳失敗訊息，status code 設 401。
        
        ```json
        {
          "success": false,
          "error": "Invalid or expired Google ID Token"
        }
        ```
        

---

## 四、n8n Workflow 設計圖

```text
[Webhook (POST /google-login)]
      │
      ▼
[HTTP Request (GET https://oauth2.googleapis.com/tokeninfo?id_token={{id_token}})]
      │
      ▼
[IF (aud/iss/exp 合法?)]
   │              │
   ▼              ▼
[True]         [False]
   │              │
   ▼              ▼
[Respond:   [Respond:
  成功]       失敗/401]
```

---

## 五、重點與常見問題

### 1. n8n 憑什麼能驗證 Google Token？

- 因為 Google 官方 TokenInfo API 會幫你驗證簽章、有效期與目標應用（audience）等資訊，不需後端額外寫驗簽程式。
    

### 2. Google ID Token 會過期嗎？

- 會，通常一小時。過期時 workflow 會自動判斷 `exp`，回傳失敗，前端收到 401 就能跳回重新登入。
    

### 3. Client ID 要不要硬寫？

- 建議存在 n8n 環境變數，用 IF 條件 `{{ $env["GOOGLE_CLIENT_ID"] }}` 來比對，更安全也方便不同環境切換。
    

### 4. 可以加更多自動化嗎？

- 當然！通過驗證後可串接 DB 寫入、郵件通知、Slack 訊息、Google Sheet、Webhook 到第三方服務…都沒問題。
    

### 5. 我可以不用寫任何 Node.js/Express 嗎？

- 是的，全部驗證與回應都交給 n8n workflow 完成！
    

---

## 六、總結

透過 n8n 的 Webhook、HTTP Request、IF 與 Respond to Webhook 節點，你能「零程式碼」完成 Google Login 的安全驗證流程。  
不論你是剛開始玩 OAuth，還是追求無後端、低維運的身分驗證解決方案，這套做法都能極速上線並保有高安全性！

---

**延伸閱讀：**

- [Google ID Token 驗證官方說明](https://developers.google.com/identity/sign-in/web/backend-auth)
    
- [n8n 官方文件：Webhook、HTTP Request、IF 節點使用教學](https://docs.n8n.io/)
    

---

這樣，你只要專心開發前端互動、n8n Workflow 自動化，登入驗證安全與便利就一次搞定！