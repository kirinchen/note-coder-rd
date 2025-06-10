

## 一、概覽

在單頁式應用（SPA）時代，前後端分離已是主流。其核心挑戰在於：**前端如何讓使用者方便登入？後端又如何確認這個登入請求是真實可信？**  
本文以 Google OAuth2/OpenID Connect（OIDC）為例，示範如何在 React 前端取得 ID Token，並在後端（Node.js/Express 或 n8n Workflow）進行簽章與 claims 驗證，最終完成「安全可信的登入驗證」。

---

## 二、OAuth2 vs. OIDC 簡介

- **OAuth2**：定義授權（authorization）流程，讓第三方應用獲得存取資源的權限（Access Token）。
    
- **OpenID Connect (OIDC)**：建立在 OAuth2 之上，引入 **ID Token**（JWT 格式），用來「驗證使用者身分」。
    

對於僅需「社群帳號登入」的場景，只需在前端拿到 Google 回傳的 ID Token，再交給後端驗證即可。Access Token 則僅在你要呼叫 Google API（如 Calendar、Drive）時才需要。

---

## 三、前端實作：取得 Google ID Token

1. **安裝並初始化 Google Identity Services**
    
    ```bash
    npm install @react-oauth/google
    ```
    
    ```jsx
    import { GoogleOAuthProvider, useGoogleLogin } from '@react-oauth/google';
    
    function App() {
      return (
        <GoogleOAuthProvider clientId={process.env.REACT_APP_GOOGLE_CLIENT_ID}>
          <LoginButton />
        </GoogleOAuthProvider>
      );
    }
    ```
    
2. **取得憑證 (credential)**
    
    ```jsx
    function LoginButton() {
      const login = useGoogleLogin({
        onSuccess: tokenResponse => {
          // tokenResponse.credential 就是 ID Token（JWT 字串）
          fetch('/api/auth/google', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ id_token: tokenResponse.credential }),
          }).then(/* 處理後端回應 */);
        },
        onError: () => alert('登入失敗'),
        flow: 'implicit', // 或 'auth-code' 依需求選擇
      });
    
      return <button onClick={() => login()}>使用 Google 登入</button>;
    }
    ```
    

> **重點**：在前端取得的 `id_token`，必須以 `POST`（或 `Authorization: Bearer <id_token>`）送到後端驗證，不可僅在前端自行信任。

---

## 四、後端驗證：確認 ID Token 真實可信

### 方法一：Node.js + google-auth-library

1. 安裝套件
    
    ```bash
    npm install google-auth-library express
    ```
    
2. 中介器 (`verifyGoogleToken`)
    
    ```javascript
    const { OAuth2Client } = require('google-auth-library');
    const client = new OAuth2Client(process.env.GOOGLE_CLIENT_ID);
    
    async function verifyGoogleToken(req, res, next) {
      const idToken = req.body.id_token || req.headers.authorization?.split(' ')[1];
      if (!idToken) return res.status(401).send('Missing ID token');
    
      try {
        const ticket = await client.verifyIdToken({
          idToken,
          audience: process.env.GOOGLE_CLIENT_ID,
        });
        const payload = ticket.getPayload();
        // 可選擇進一步檢查 payload.iss、payload.exp、payload.email_verified 等
        req.user = {
          uid: payload.sub,
          email: payload.email,
          name: payload.name,
        };
        next();
      } catch (err) {
        console.error(err);
        res.status(401).send('Invalid or expired ID token');
      }
    }
    ```
    
3. 應用於路由
    
    ```javascript
    const express = require('express');
    const app = express();
    app.use(express.json());
    
    app.post('/api/auth/google', verifyGoogleToken, (req, res) => {
      // 驗證通過，執行登入或註冊邏輯
      res.json({ success: true, user: req.user });
    });
    
    app.listen(3000, () => console.log('Server on port 3000'));
    ```
    

---

### 方法二：呼叫 Google TokenInfo 端點

若不想額外安裝 SDK，只要把 `id_token` 傳給：

```
GET https://oauth2.googleapis.com/tokeninfo?id_token=<ID_TOKEN>
```

Google 回傳 JSON claims，狀態碼 200 表示有效；否則回 400/401。你可在後端用任何語言的 HTTP 客戶端完成這步，並自行檢查 `aud`、`iss`、`exp` 等欄位。

---

### 方法三：在 n8n Workflow 中驗證

如果你的後端完全由 n8n 承擔，流程示意：

1. **Webhook 節點** (`POST /google-login`) 接收 `{ id_token }`
    
2. **HTTP Request 節點** 呼叫 `https://oauth2.googleapis.com/tokeninfo?id_token={{$json["id_token"]}}`
    
3. **IF 節點** 驗證：
    
    - `aud === $env.GOOGLE_CLIENT_ID`
        
    - `iss === "accounts.google.com"`
        
    - `exp > Math.floor(Date.now()/1000)`
        
4. **合法分支**：繼續業務（寫 DB、回傳 Success）；**不合法分支**：使用 `Respond to Webhook` 回 401
    

這種無碼方式，不需自架後端服務也能快速驗證。

---

## 五、安全注意事項與進階

1. **HTTPS**：務必透過 TLS，加密前後端與 n8n 之間的通訊。
    
2. **Token 過期**：Google ID Token 有效期約一小時，前端需在過期前重新取得。
    
3. **後端 Session 管理**：若不想每次請求都驗 `id_token`，可在驗證通過後，自行簽發短效 JWT 或設置 HttpOnly Cookie，後續請求只需帶此憑證。
    
4. **最小授權原則**：前端只請求 `openid email profile` scope，即可取得基本登入所需資訊，避免濫用過多權限。
    

---

## 六、小結

- **前端**：透過 Google Identity Services 拿到 `id_token`，並安全地送到後端。
    
- **後端**：以 JWT 驗證（google-auth-library）或呼叫 TokenInfo，檢查 `aud`、`iss`、`exp`、`email_verified` 等，確保 Token 真實有效。
    
- **n8n 無碼方案**：也能簡單串出完整驗證流程，快速完成「社群帳號登入」功能。
    

掌握上述流程，即可輕鬆為 React 或其他 SPA 應用串接 Google OAuth2/OIDC，並透過後端嚴謹驗證，打造安全可信的登入系統。