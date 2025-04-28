

# 🚀 PGVector 安裝與使用教學｜超快懶人包版（含 AWS RDS PostgreSQL）

## ✨ PGVector 是什麼？

- PostgreSQL 的向量資料擴充套件
    
- 讓你直接在資料庫中儲存 Embedding、做向量相似度搜尋（支援 L2、Cosine）
    
- 適用場景：AI 搜尋、RAG 系統、推薦系統
    

---

## 🛠 安裝 PGVector

### 如果是自架 PostgreSQL

1. 安裝：
    
    ```bash
    sudo apt install postgresql-server-dev-14  # 依你版本
    git clone https://github.com/pgvector/pgvector.git
    cd pgvector
    make
    sudo make install
    ```
    
2. 在資料庫執行：
    
    ```sql
    CREATE EXTENSION IF NOT EXISTS vector;
    ```
    

---

### 如果是 AWS RDS PostgreSQL / Aurora

✅ AWS 已原生支援 pgvector（無需安裝套件）  
版本要求：

- PostgreSQL 15.2+ / 14.7+ / 13.10+ / 12.14+
    
- Aurora PostgreSQL 15.3+ / 14.8+ / 13.11+ / 12.15+
    

只需連接資料庫，執行：

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

---

## 📦 建立向量表格範例

```sql
CREATE TABLE documents (
    id VARCHAR(50) PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536)  -- 以 OpenAI Ada 模型為例
);
```

---

## 🔎 查詢向量相似度

以 Cosine 相似度排序：

```sql
SELECT id, content
FROM documents
ORDER BY embedding <=> ARRAY[0.1, 0.2, 0.3, ..., 0.1536]
LIMIT 5;
```

- `<=>`：Cosine 相似度
    
- `<->`：L2 距離
    

---

## ⚡ 加速搜尋（建立 HNSW Index）

建議加速大資料集搜尋：

```sql
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```

---

## 🐍 Python 快速串接範例（psycopg3）

```python
import psycopg
import openai

openai.api_key = "your-api-key"

def generate_embedding(text):
    response = openai.embeddings.create(
        input=text,
        model="text-embedding-ada-002"
    )
    return response.data[0].embedding

with psycopg.connect("postgresql://user:password@host:5432/dbname") as conn:
    with conn.cursor() as cur:
        embedding = generate_embedding("這是一段測試文字")
        cur.execute(
            "INSERT INTO documents (id, content, embedding) VALUES (%s, %s, %s)",
            ("your_id", "這是一段測試文字", embedding)
        )
    conn.commit()
```

---

# 🎯 小結

|特色|說明|
|:--|:--|
|整合性高|在原有 PostgreSQL 資料庫直接操作|
|相容性好|支援標準 SQL 語法|
|雲端友善|AWS RDS / Aurora PostgreSQL 完美支援|
|效能擴充|可用 HNSW Index 加速近似最近鄰搜尋|

---

# 📚 延伸應用

- 文件語意搜尋器
    
- AI 知識庫（RAG）
    
- 產品推薦系統
    
- 聊天機器人記憶庫
    

---

