#### 🟢 Template A: Java + Spring Boot (API 與商業邏輯)

**適用場景**：提供給前端的高併發 API、交易系統後端。

**Constitution 亮點**：

-   **技術限制**：僅使用 Java 21+ 與 Spring Boot 3.5.*。嚴格採用 Controller -> Service -> Repository 三層架構。
    
-   **測試規範 (Testable)**：所有 Service 層的商業邏輯必須搭配 JUnit 5 與 Mockito 進行單元測試。MVP 階段不強制寫 E2E 測試。
    
-   **API 契約**：強制所有 API 開發完成後，必須透過 Springdoc 自動產生 OpenAPI 3.0 規格文件。
    
