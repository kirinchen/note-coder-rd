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
    

