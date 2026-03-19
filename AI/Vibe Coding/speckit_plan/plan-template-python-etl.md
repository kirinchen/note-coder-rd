#### 🔵 Template B: Python 12+ (ETL、資料分析與 AI)

**適用場景**：處理企業的 ESG 永續數據清洗管線、量化交易的特徵工程與演算法模型。

**Constitution 亮點**：

-   **技術限制**：僅使用 Python 3.12+。明確規範依賴管理（如使用 `uv` 或 `requirements.txt`），並針對數據處理鎖定套件（如 Pandas, Scikit-learn, XGBoost）。
    
-   **架構精神**：以 Functional Programming 或清晰的 Pipeline 架構為主。資料的萃取 (Extract)、轉換 (Transform) 與載入 (Load) 必須拆分為獨立可測試的模組。
    
-   **測試規範 (Testable)**：強制使用 Pytest。針對資料轉換邏輯，必須提供 Mock Data 的測試案例。
    
-   **防呆**：如果需要提供對外介面，僅允許使用輕量級的 FastAPI，嚴禁引入 Django 等過度龐大的全端框架。