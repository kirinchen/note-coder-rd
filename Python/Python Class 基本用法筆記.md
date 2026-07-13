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