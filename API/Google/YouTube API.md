
# YouTube Data API v3 上傳影片（Resumable Upload）

在網路環境不穩或影片檔案較大時，YouTube 提供了 **Resumable Upload（可續傳上傳）** 功能，幫助開發者穩定且安全地將影片上傳至指定頻道。本文將重點介紹：

- 什麼是 `uploadType=resumable`？
    
- 上傳流程說明
    
- 特別說明：**如何在多頻道管理（Content Owner）情境下指定頻道上傳**
    

---

## 什麼是 `uploadType=resumable`？

當您呼叫以下端點時：

```
POST https://www.googleapis.com/upload/youtube/v3/videos?part=snippet,status&uploadType=resumable
```

您可以啟動一個可續傳的上傳流程，這種方式有以下特點：

- **避免中斷失敗**：若網路中斷，可以從上一次成功上傳的進度繼續，無需重來。
    
- **適合大檔案**：建議使用於超過 10 MB 的影片檔案。
    
- **更細緻的控制**：可以根據需要自行切割與管理上傳的資料區塊（chunks）。
    

---

## 上傳流程概覽

1. **建立上傳會話**
    
    - 發送 `POST` 請求，帶上影片的 `snippet`（標題、描述、標籤等）與 `status`（隱私設定，如公開、私人或不公開）。
        
    - 成功後，伺服器會回傳一個 `Location`，指向專屬的上傳 URL。
        
2. **上傳影片資料**
    
    - 使用 `PUT` 請求將影片檔案（可分段）傳送至上一步提供的上傳 URL。
        
    - 每次上傳都需要帶上 `Content-Range` 標頭來標示這次傳送的範圍。
        
3. **完成與確認**
    
    - 最後一段上傳完成後，YouTube 會回傳完整的影片資源資料。
        

---

## 例外重點：**內容合作夥伴（Content Owner）如何指定頻道？**

一般情況下，影片會上傳到您 **OAuth 2.0 認證的帳號所對應的 YouTube 頻道**。  
**無法直接指定其他 `channelId`。**

但如果您是屬於 **內容合作夥伴（Content Owner / CMS）**，管理多個 YouTube 頻道時，  
您可以透過以下額外參數，把影片指定上傳到特定子頻道：

- `onBehalfOfContentOwner=<內容合作夥伴 ID>`
    
- `onBehalfOfContentOwnerChannel=<目標子頻道 ID>`
    

> ✅ **這是唯一能「指定頻道上傳」的方法。**  
> ✅ **必須是正式的 YouTube 內容合作夥伴（CMS）帳戶，才可使用這兩個參數。**

使用這種方式，內容合作夥伴可以在一個 OAuth 認證之下，為旗下管理的不同頻道各自上傳影片，不需為每個頻道個別授權，大幅簡化操作與管理流程。

---

## 小結

使用 `uploadType=resumable` 可以大幅提高影片上傳的穩定性與可靠性，是開發大型上傳功能時的最佳選擇。  
若您是內容合作夥伴（CMS），更可以透過 `onBehalfOfContentOwner` 與 `onBehalfOfContentOwnerChannel`，靈活將影片發佈至旗下管理的任意子頻道。

掌握這些技巧，讓您的 YouTube 上傳流程更穩定、更有彈性！

