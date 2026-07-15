# Python Class 基本用法筆記

```python
class Dog:
    # 類別屬性（所有實例共享）
    species = "Canis familiaris"

    # 建構子（初始化實例）
    def __init__(self, name, age):
        self.name = name      # 實例屬性
        self.age = age

    # 實例方法
    def bark(self):
        return f"{self.name} 汪汪叫！"

    # 使用實例屬性
    def info(self):
        return f"{self.name} 今年 {self.age} 歲"


# 建立實例
d = Dog("小黑", 3)
print(d.bark())        # 小黑 汪汪叫！
print(d.info())        # 小黑 今年 3 歲
print(d.species)       # Canis familiaris
```

## 核心概念

**`self`**：指向實例本身，方法第一個參數必須是它。

**類別屬性 vs 實例屬性**

```python
Dog.species          # 透過類別存取
d.species            # 透過實例存取（找不到實例屬性時往上找類別）
d.name               # 實例屬性，各實例獨立
```

## 繼承

```python
class Puppy(Dog):
    def __init__(self, name, age, owner):
        super().__init__(name, age)   # 呼叫父類建構子
        self.owner = owner

    def bark(self):                    # 覆寫（override）
        return f"{self.name} 嗚嗚～"

p = Puppy("阿福", 1, "Kirin")
print(p.bark())        # 阿福 嗚嗚～
print(p.info())        # 繼承自 Dog
print(isinstance(p, Dog))   # True
```

## 特殊方法（Dunder Methods）

```python
class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):                # 給開發者看（除錯）
        return f"Point({self.x}, {self.y})"

    def __str__(self):                 # 給使用者看（print）
        return f"({self.x}, {self.y})"

    def __eq__(self, other):           # == 比較
        return self.x == other.x and self.y == other.y

    def __add__(self, other):          # + 運算
        return Point(self.x + other.x, self.y + other.y)


p1 = Point(1, 2)
p2 = Point(3, 4)
print(p1 + p2)         # (4, 6)
print(p1 == Point(1, 2))   # True
print(repr(p1))        # Point(1, 2)
```

## 屬性封裝

```python
class Account:
    def __init__(self, balance):
        self._balance = balance        # 慣例：單底線 = 內部使用
        self.__secret = "私有"          # 雙底線 = name mangling

    @property                          # 讓方法像屬性一樣存取
    def balance(self):
        return self._balance

    @balance.setter                    # 設定時驗證
    def balance(self, value):
        if value < 0:
            raise ValueError("餘額不能為負")
        self._balance = value


a = Account(100)
print(a.balance)       # 100（不用括號）
a.balance = 200        # 走 setter
```

## 類別方法與靜態方法

```python
class Circle:
    pi = 3.14159

    def __init__(self, r):
        self.r = r

    @classmethod                       # 第一參數是 cls，常用於替代建構子
    def unit(cls):
        return cls(1)

    @staticmethod                      # 不需要 self / cls，純工具函式
    def is_valid(r):
        return r > 0

c = Circle.unit()          # 建立半徑 1 的圓
Circle.is_valid(5)         # True
```

## 常用速查

|語法|用途|
|---|---|
|`__init__`|建構子|
|`self`|實例參照|
|`super().__init__()`|呼叫父類|
|`@property`|getter|
|`@classmethod`|類別方法（`cls`）|
|`@staticmethod`|靜態方法|
|`isinstance(x, Cls)`|型別檢查|
|`@dataclass`|自動產生 `__init__`/`__repr__`（見下）|

## 現代寫法：dataclass

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int = 0                       # 有預設值

    def greet(self):
        return f"Hi, {self.name}"

u = User("Kirin", 30)
print(u)               # User(name='Kirin', age=30) 自動 __repr__
```


# Python ABC（抽象基底類別）

```python
from abc import ABC, abstractmethod

class Animal(ABC):                      # 繼承 ABC
    @abstractmethod
    def sound(self):                    # 抽象方法：子類「必須」實作
        ...

    def describe(self):                 # 一般方法：可直接繼承
        return f"我發出 {self.sound()} 的聲音"


# 不能實例化抽象類別
# Animal()  → TypeError: Can't instantiate abstract class

class Dog(Animal):
    def sound(self):                    # 必須實作，否則 Dog 也不能實例化
        return "汪汪"

d = Dog()
print(d.sound())        # 汪汪
print(d.describe())     # 我發出 汪汪 的聲音
```

**核心規則**：只要類別有任一未實作的 `@abstractmethod`，就無法被實例化，強制子類補齊介面。

## 抽象屬性與其他組合

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @property
    @abstractmethod                     # 抽象屬性（順序：property 在外）
    def area(self):
        ...

    @classmethod
    @abstractmethod                     # 抽象類別方法
    def from_config(cls, cfg):
        ...

    @staticmethod
    @abstractmethod                     # 抽象靜態方法
    def unit():
        ...


class Circle(Shape):
    def __init__(self, r):
        self.r = r

    @property
    def area(self):
        return 3.14159 * self.r ** 2

    @classmethod
    def from_config(cls, cfg):
        return cls(cfg["r"])

    @staticmethod
    def unit():
        return Circle(1)


c = Circle(2)
print(c.area)           # 12.56636
```

## 抽象方法可含預設邏輯（子類用 super）

```python
class Base(ABC):
    @abstractmethod
    def load(self):
        # 抽象方法仍可有實作，供子類 super() 呼叫
        return "共用前處理"

class Impl(Base):
    def load(self):
        prefix = super().load()         # 呼叫父類邏輯
        return f"{prefix} + 自訂"
```

## register：虛擬子類（不繼承也算子類）

```python
from abc import ABC

class Serializable(ABC):
    ...

class LegacyJSON:                       # 沒繼承 Serializable
    def to_json(self): ...

Serializable.register(LegacyJSON)       # 註冊為虛擬子類

print(issubclass(LegacyJSON, Serializable))  # True
print(isinstance(LegacyJSON(), Serializable)) # True
# 注意：register 不強制實作方法，只影響 isinstance/issubclass 判斷
```

## **subclasshook**：鴨子型別檢查

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self): ...

    @classmethod
    def __subclasshook__(cls, C):
        if cls is Drawable:
            # 只要有 draw 方法就算 Drawable（不需顯式繼承）
            if any("draw" in B.__dict__ for B in C.__mro__):
                return True
        return NotImplemented

class Widget:
    def draw(self): ...

print(issubclass(Widget, Drawable))     # True
```

## ABC vs ABCMeta

```python
from abc import ABC, ABCMeta

# 這兩種寫法等價：
class A(ABC): ...
class B(metaclass=ABCMeta): ...
# 若已有自訂 metaclass，需用 ABCMeta 合併；否則直接繼承 ABC 較簡潔
```

## Protocol：結構型別（常用替代方案）

適合純介面約束、不想強制繼承時，比 ABC 更貼近鴨子型別：

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Sized(Protocol):
    def __len__(self) -> int: ...       # 只定義形狀，不需實作

class Box:
    def __len__(self): return 5

def show(x: Sized):                     # 靜態檢查 + 提示
    print(len(x))

show(Box())                             # OK，Box 結構相符
print(isinstance(Box(), Sized))         # True（需 runtime_checkable）
```

## ABC vs Protocol 選型

|情境|選擇|
|---|---|
|要強制子類實作、共用基底邏輯|**ABC**|
|有明確繼承階層|**ABC**|
|只約束「形狀」、不想綁繼承|**Protocol**|
|第三方類別無法改繼承|**Protocol** 或 `register`|
|需要 `super()` 共用實作|**ABC**|

## 常用速查

|語法|用途|
|---|---|
|`class X(ABC)`|宣告抽象類別|
|`@abstractmethod`|標記必須實作的方法|
|`@property` + `@abstractmethod`|抽象屬性（property 在外層）|
|`X.register(Cls)`|註冊虛擬子類|
|`__subclasshook__`|自訂 issubclass 判斷|
|`metaclass=ABCMeta`|需合併自訂 metaclass 時|
|`typing.Protocol`|結構型別，免繼承約束|