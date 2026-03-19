
#### 🟠 Template C: Frontend (React + Vite)

**適用場景**：數據儀表板、內部管理後台介面。

**Constitution 亮點**：

-   **技術限制**：React + Vite + TypeScript + Tailwind CSS。
    
-   **架構精神**：純粹的 Client-side 應用。狀態管理保持輕量（如使用 Zustand 或 React Query），避免過度依賴 Redux。
    
-   **測試規範 (Testable)**：核心 UI 元件與自訂 Hook 必須使用 Vitest 撰寫測試。
    
-   **防呆**：前端只負責「展示數據」與「發送 API」，嚴禁在前端實作任何複雜的商業運算或數據過濾邏輯。
    