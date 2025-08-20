

### 為什麼備份如此重要？

即使 Docker Volume 的機制非常可靠，我們依然強烈建議您進行手動備份，原因如下：

- **人為錯誤：** 您或團隊成員可能會意外刪除重要的工作流程或資料，備份是唯一的救星。
    
- **硬體故障：** 伺服器的硬碟可能會損壞，導致 Docker Volume 遺失。
    
- **升級風險：** 在進行 n8n 或 PostgreSQL 的重大版本升級時，雖然通常是安全的，但備份能確保一旦出現問題，您有路可退。
    
- **資料遷移：** 當您需要將 n8n 服務遷移到另一台伺服器時，備份與還原是最標準的作法。
    

簡單來說，**備份是您資料的「保險」**。

---

## 第一部分：詳細備份步驟 (`pg_dump`)

`pg_dump` 是 PostgreSQL 官方提供的邏輯備份工具。它會將您的資料庫結構 (Schema) 和資料內容轉換成一個 `.sql` 格式的純文字檔。這個檔案包含了所有重建資料庫所需的 SQL 指令。

### 指令詳解

讓我們來拆解這個指令的每一部分，讓您完全理解它在做什麼：

Bash

```
docker-compose exec -T postgres pg_dump -U ${POSTGRES_USER} -d ${POSTGRES_DB} > my_backup_20250820.sql
```

- `docker-compose exec`: 這是 `docker-compose` 的指令，意思是「在指定的服務容器內執行一個命令」。
    
- `-T`: 這是一個很有用的參數，它會停用 TTY (虛擬終端機)，讓指令在背景執行更穩定，特別是當您需要將輸出重新導向到檔案時，可以避免一些不必要的格式問題。
    
- `postgres`: 這是您在 `docker-compose.yml` 中定義的 PostgreSQL 服務名稱 (`service name`)。
    
- `pg_dump`: 這是真正執行的備份程式，它運行在 `postgres` 容器內部。
    
- `-U ${POSTGRES_USER}`: 指定用哪個使用者身份來執行備份。`${POSTGRES_USER}` 會讀取您 `.env` 檔案中定義的 PostgreSQL 超級使用者名稱。使用超級使用者可以確保您有權限讀取所有資料。
    
- `-d ${POSTGRES_DB}`: 指定要備份的資料庫名稱。`${POSTGRES_DB}` 會讀取您 `.env` 檔案中定義的資料庫名稱。
    
- `>`: 這是 Linux/macOS/Windows 中標準的「輸出重新導向」符號。它會將左邊指令 (`pg_dump`) 產生的所有內容，寫入到右邊的檔案中。
    
- `my_backup_20250820.sql`: 這是您在**主機 (Host Machine)** 上儲存備份的檔案名稱。建議您加上日期，方便管理。**請注意：這個檔案是建立在您的電腦上，而不是在 Docker 容器裡**，這正是我們想要的。
    

### 如何執行備份

1. 開啟您的終端機 (Terminal)。
    
2. 使用 `cd` 指令切換到包含 `docker-compose.yml` 和 `.env` 檔案的專案目錄。
    
3. 確認您的 n8n 服務正在運行 (`docker-compose ps` 應該能看到 `postgres` 服務是 `up` 的狀態)。
    
4. 複製並貼上完整的備份指令，然後按下 Enter：
    
    Bash
    
    ```
    docker-compose exec -T postgres pg_dump -U ${POSTGRES_USER} -d ${POSTGRES_DB} > my_backup_$(date +%Y%m%d).sql
    ```
    
    _(這裡我用 `$(date +%Y%m%d)` 來自動產生當天的日期，例如 `my_backup_20250820.sql`)_
    
5. **驗證備份：**
    
    - 執行 `ls -l` 指令，您應該會看到剛剛建立的 `.sql` 檔案。
        
    - 您可以用文字編輯器打開這個檔案，會看到裡面充滿了 `CREATE TABLE...` 和 `COPY ... FROM stdin;` 等 SQL 指令，這代表備份成功。
        

---

## 第二部分：詳細還原步驟 (`psql`)

還原通常是在一個全新的、空的資料庫上進行的，以避免新舊資料衝突。`psql` 是 PostgreSQL 的命令列客戶端，它可以執行 `.sql` 檔案中的指令。

### 何時需要還原？

- 當您想從災難中恢復資料時。
    
- 當您將 n8n 遷移到一台新伺服器時。
    
- 當您想建立一個用於測試的、和正式環境一模一樣的副本時。
    

### 還原前的準備：建立一個乾淨的環境

**警告：以下操作會徹底刪除您目前的資料庫資料。請務必在確認已有可靠備份後才執行。**

1. **停止並移除所有服務容器：**
    
    Bash
    
    ```
    docker-compose down
    ```
    
2. **刪除舊的資料庫儲存卷：**
    
    - 首先，執行 `docker volume ls` 查看您所有的儲存卷，找到與專案相關的資料庫儲存卷 (通常是 `專案目錄名稱_db_storage`)。
        
    - 執行刪除指令，例如如果您的專案目錄是 `my-n8n-project`，指令就是：
        
        Bash
        
        ```
        docker volume rm my-n8n-project_db_storage
        ```
        
3. **重新啟動服務：**
    
    Bash
    
    ```
    docker-compose up -d
    ```
    
    這時 Docker 會因為找不到 `db_storage` 儲存卷而建立一個全新的、空的儲存卷，並初始化一個全新的、空的 PostgreSQL 資料庫。
    

### 指令詳解

Bash

```
docker-compose exec -T postgres psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} < my_backup_20250820.sql
```

- `docker-compose exec -T postgres psql`: 大部分和備份指令類似，只是執行的程式從 `pg_dump` 變成了 `psql`。
    
- `-U ${POSTGRES_USER}` 和 `-d ${POSTGRES_DB}`: 指定要還原到哪個資料庫以及使用哪個使用者身份。
    
- `<`: 這是「輸入重新導向」符號，和 `>` 正好相反。它會將右邊檔案 (`my_backup_20250820.sql`) 的內容，作為左邊指令 (`psql`) 的輸入。
    

### 如何執行還原

1. 確保您的終端機位於 `docker-compose.yml` 和備份檔案 (`.sql`) 所在的目錄。
    
2. 確保您已經完成了**還原前的準備**，擁有一個全新的空資料庫。
    
3. 複製並貼上完整的還原指令，然後按下 Enter：
    
    Bash
    
    ```
    docker-compose exec -T postgres psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} < my_backup_20250820.sql
    ```
    
    _(請將檔名換成您實際的備份檔名)_
    
4. **驗證還原：**
    
    - 連線到資料庫：
        
        Bash
        
        ```
        docker-compose exec postgres psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}
        ```
        
    - 在 psql 命令列中，輸入 `\dt` 並按下 Enter。如果您能看到許多 n8n 相關的表格 (例如 `workflow_entity`, `execution_entity` 等)，就代表您的資料已經成功還原了！
        
    - 輸入 `\q` 離開 psql。
        

現在，您的 n8n 應該已經恢復到您備份時的狀態了。