## 告別冗長程式碼：Python @dataclass 必知的 5 個實用技巧

在 Python 3.7 引入 `@dataclass` 之前，我們定義一個單純用來儲存資料的類別（Class），往往需要寫一堆重複的 `__init__`、`__repr__` 甚至是 `__eq__`。`@dataclass` 的出現徹底解決了這個痛點。

除了基本的用法外，這裡整理了 5 個開發者最常用、也最容易踩坑的進階技巧，讓你的程式碼更 Pythonic！

### 1. 善用 `__post_init__` 進行驗證與後處理

`@dataclass` 最強大的地方在於它保留了客製化的彈性。當你需要驗證傳入的參數，或是某個欄位是由其他欄位組合而成時，不需要自己重寫 `__init__`，只要定義 `__post_init__` 即可。



```Python
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False) # 不需傳入，自動計算

    def __post_init__(self):
        if self.width <= 0 or self.height <= 0:
            raise ValueError("寬高必須大於 0")
        self.area = self.width * self.height
```

### 2. 小心 Mutable Default Trap！使用 `default_factory`

這是新手最容易踩到的地雷。如果你想給 List 或 Dict 預設值，**絕對不能**直接寫 `items: list = []`。這會導致所有物件實例共用同一個 List（這是 Python 的特性）。

**正確做法**是使用 `default_factory`：



```Python
from dataclasses import dataclass, field
from typing import List

@dataclass
class ShoppingCart:
    # ❌ 錯誤: items: List[str] = []
    # ✅ 正確:
    items: List[str] = field(default_factory=list)

cart1 = ShoppingCart()
cart1.items.append("Apple")

cart2 = ShoppingCart()
# cart2.items 依然是空的，不會包含 "Apple"
```

### 3. 打造不可變物件：`frozen=True`

在 Functional Programming 或多執行緒環境中，我們常希望物件建立後就不能被修改。只要加上 `frozen=True`，你的 Data Class 就會變成 Immutable（不可變），而且自動獲得 `__hash__` 方法，這意味著它可以被當作 Dictionary 的 Key！



```Python
@dataclass(frozen=True)
class Config:
    host: str
    port: int

conf = Config("localhost", 8080)
# conf.port = 9090  # 這行會報錯：FrozenInstanceError

# 可以當作 dict key
settings = {conf: "Production Server"}
```

### 4. 快速轉型：`asdict` 與 `astuple`

在做 API 開發時，我們常需要把物件轉成 JSON 格式。`dataclasses` 模組內建了轉換工具，讓你不需要手寫 Serializer。



```Python
from dataclasses import asdict

@dataclass
class User:
    id: int
    name: str

user = User(1, "Alice")
user_dict = asdict(user) 
# 輸出: {'id': 1, 'name': 'Alice'} -> 可以直接丟進 json.dumps()
```

### 5. 隱藏敏感資訊：`field(repr=False)`

當我們 print 一個物件時，`@dataclass` 會自動幫我們把所有欄位印出來。但如果裡面包含密碼或金鑰，這會造成資安風險。使用 `repr=False` 可以讓該欄位在 print 時隱形。



```Python
@dataclass
class Database:
    uri: str
    password: str = field(repr=False)

db = Database("mysql://...", "secret123")
print(db) 
# 輸出: Database(uri='mysql://...') -> 密碼不會被印出來
```


### 使用 `field(init=False)` 處理內部私有成員或服務

有時候，我們的 Data Class 需要持有某些「工具物件」（如資料庫連線、API Client 或 Service 類別），但我們不希望這些物件出現在 `__init__` 的參數列表中。

如果直接寫 `service: MyService`，Python 會要求你在實例化時傳入它。這時，`field(init=False)` 就是你的救星：



```Python
from dataclasses import dataclass, field

@dataclass
class TaskTraining:
    task_name: str
    # 宣告型別以獲得 IDE 提示，但設定 init=False 讓它從建構子消失
    ctx_service: PipelineContextService = field(init=False)

    def __post_init__(self):
        # 在這裡進行內部的初始化
        self.ctx_service = PipelineContextService()
```

**為什麼要這樣寫？**

- **介面簡潔**：調用者只需要寫 `TaskTraining(task_name="Model_A")`，不需要關心內部的 Service 如何建立。
    
- **型別安全**：透過宣告 `ctx_service: PipelineContextService`，你在撰寫程式碼時，IDE 能準確提供該 Service 的方法提示。
    
- **職責分離**：確保資料的定義與功能的執行邏輯（Service）被清晰地隔離開來。
    
這是一篇專為 Python 開發者撰寫的技術部落格文章，整理了你剛剛遇到的 `dataclass` 繼承問題、原理分析以及最佳解法。你可以直接發布在你的 Medium、技術 Blog 或團隊內部的知識庫中。

---

# Python Dataclass 繼承完全指南：避開 `TypeError` 的陷阱與最佳實踐

Python 的 `@dataclass` 是自 3.7 引入以來最受歡迎的語法糖之一。它讓我們免於撰寫冗長的 `__init__`、`__repr__` 和 `__eq__`，讓程式碼變得乾淨優雅。

然而，當我們開始將 **Dataclass** 與 **繼承 (Inheritance)** 結合使用，特別是涉及到「預設值 (Default Values)」時，往往會撞上一堵牆。

這篇文章將帶你深入探討 Dataclass 的繼承機制，解析為什麼會出現 `Non-default argument follows default argument` 錯誤，以及如何優雅地解決它。

## 1. 災難現場：當子類別遇到父類別的預設值

假設我們正在設計一個資料處理流程（Pipeline），我們有一個基底類別 `Provider`，以及一個繼承它的具體實作 `FileProvider`。



```Python
from dataclasses import dataclass

@dataclass
class Provider:
    # 父類別有一個有預設值的欄位
    timeout: int = 30 

@dataclass
class FileProvider(Provider):
    # 子類別有一個「沒有」預設值的必填欄位
    path: str 
```

這段程式碼看起來合情合理，但在執行時，Python 會無情地拋出錯誤：

> **TypeError: non-default argument follows default argument**

### 為什麼會這樣？

這不是 Dataclass 的 bug，而是 Python 函式定義的基本規則：**「有預設值的參數，必須放在沒有預設值的參數後面」**。

當 `@dataclass` 幫我們生成 `__init__` 方法時，它會依照 **MRO (Method Resolution Order)** 順序，先收集父類別的欄位，再收集子類別的欄位。

生成的 `__init__` 大致長這樣：



```Python
# 這是 Python 語法不允許的！
def __init__(self, timeout=30, path): 
    self.timeout = timeout
    self.path = path
```

因為 `timeout` (有預設值) 排在 `path` (無預設值) 前面，導致語法錯誤。

---

## 2. 解法一：使用 `kw_only=True` (Python 3.10+ 推薦 👑)

在 Python 3.10 之後，官方終於給出了最完美的解決方案：`kw_only=True`。

這個參數會強制所有生成的 `__init__` 參數都必須使用 **關鍵字引數 (Keyword Arguments)** 傳入。這意味著參數的順序不再重要，因為你必須明確指定參數名稱。

### 如何修改？

只要在父類別和子類別的裝飾器中加上 `kw_only=True`：



```Python
from dataclasses import dataclass

@dataclass(kw_only=True)
class Provider:
    timeout: int = 30

@dataclass(kw_only=True)
class FileProvider(Provider):
    path: str 

# ✅ 成功！
# 注意：實例化時必須寫出參數名稱
p = FileProvider(path="/tmp/data.csv", timeout=60)
```

生成的 `__init__` 會變成這樣（注意那個 `*`）：



```Python
def __init__(self, *, timeout=30, path):
    ...
```

這解決了順序問題，同時也強制了呼叫者寫出更清晰的程式碼（Explicit is better than implicit）。

---

## 3. 解法二：子類別也給預設值 (適用於舊版 Python)

如果你被困在 Python 3.9 以下，或者你不想強制使用 Keyword Arguments，唯一的解法是：**確保子類別的所有欄位也都有預設值**。

如果你希望某個欄位是「必填」的，但又被迫要給預設值，通常我們會給 `None`，然後在 `__post_init__` 裡做檢查。



```Python
@dataclass
class FileProvider(Provider):
    # 給予預設值 None 來騙過 Python 的語法檢查
    path: str = None 

    def __post_init__(self):
        if self.path is None:
            raise ValueError("path cannot be None!")
```

雖然這能動，但比起解法一，這顯然是一種 workaround，犧牲了部分型別提示的準確性。

---

## 4. 進階技巧：Dataclass 與 `__post_init__` 的繼承

解決了初始化問題後，另一個常見的坑是 `__post_init__`。

在普通的 Python Class 中，我們習慣在 `__init__` 裡呼叫 `super().__init__()`。但在 Dataclass 中，**子類別生成的 `__post_init__` 不會自動呼叫父類別的 `__post_init__`**。

如果你在父類別裡有一些初始化邏輯（例如驗證或預計算），務必在子類別手動呼叫：



```Python
@dataclass
class BaseTask:
    id: str
    def __post_init__(self):
        print(f"BaseTask init: {self.id}")

@dataclass
class ImageTask(BaseTask):
    url: str
    def __post_init__(self):
        # ⚠️ 必須手動呼叫，否則 "BaseTask init" 不會印出來
        super().__post_init__() 
        print(f"ImageTask init: {self.url}")
```

---

## 5. 總結：Dataclass 繼承的最佳實踐

在設計大型系統（如 ETL Flow、ML Pipeline）時，我們經常會用到繼承。為了避免踩雷，建議遵循以下原則：

1. **擁抱 Python 3.10+**：儘量使用新版 Python。
    
2. **預設開啟 `kw_only=True`**：在定義 Base Class 時就加上這個參數。這不僅解決了繼承順序問題，還能強迫使用者寫出可讀性更高的程式碼（`Task(id=1)` 比 `Task(1)` 清楚得多）。
    
3. **善用 ABC**：Dataclass 與 `abc.ABC` 可以完美共存。用 ABC 定義介面，用 Dataclass 實作資料結構。
    
4. **小心 `__post_init__`**：記得手動呼叫 `super()`。
    

### 範例：完美的 Dataclass 架構



```Python
from dataclasses import dataclass
from abc import ABC, abstractmethod

@dataclass(kw_only=True)
class AbstractContext(ABC):
    """基礎 Context，強制 Keyword arguments"""
    run_id: str
    debug: bool = False

@dataclass(kw_only=True)
class PipelineContext(AbstractContext):
    """具體實作，混合了有預設值與無預設值的欄位"""
    source_db: str
    retries: int = 3  # 這在普通 dataclass 繼承會報錯，但在 kw_only 下完全沒問題！

# 使用起來非常優雅
ctx = PipelineContext(
    run_id="20231027_001",
    source_db="production_db"
)
```

掌握這些技巧，你的 Python 架構將會兼具 Dataclass 的簡潔與 OOP 的彈性！
