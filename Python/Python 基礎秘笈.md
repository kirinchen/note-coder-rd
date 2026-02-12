---
title: Python Code Statement / Comment Method
tags: [python]

---

# Python Code Statement / Comment Method

## dict

### default foreach
* Code
```python=
a_dict = {'color': 'blue', 'fruit': 'apple', 'pet': 'dog'}
for key in a_dict:
    print(key)

```

* output
```bash=
color
fruit
pet
```

###  foreach .items()

* Code
```python=
a_dict = {'color': 'blue', 'fruit': 'apple', 'pet': 'dog'}
for key in a_dict.items():
    print(key)
```

* output
```bash=
('color', 'blue')
('fruit', 'apple')
('pet', 'dog')
```

* Code
```python=
for key, value in a_dict.items():
    print(key, '->', value)
```

* Output
```bash=
color -> blue
fruit -> apple
pet -> dog
```


## datatime

### 1. 基本用法

```Python
from datetime import datetime

# 設定為 2026年 2月 13日 15點 30分
dt = datetime(2026, 2, 13, 15, 30)

print(dt) 
# 輸出: 2026-02-13 15:30:00
```

### 2. 從現有時間「修改」到分

### String(ISO8601) to Datetime

```python=
import dateutil.parser

lAt: datetime = dateutil.parser.parse("2019-09-07T-15:50+00") 

# 使用 3.11+ 內建方法，效能最優 dt = 

datetime.fromisoformat(iso_text)

# 設定到「分」（抹除秒與微秒） 
dt_min = dt.replace(second=0, microsecond=0)

```

### Datetime to Unix timestamp
```python=
lAt: datetime = ...
t= lAt.timestamp()
```

### datetime to ISO8601 String

```python=
time=datetime.now()
time.isoformat()
```

### +10天
```python=
end_date = date_1 + datetime.timedelta(days=10)
```

### 兩個日期相差(天 / 秒)
```python=
day1 = datetime.now()
day12 = datetime(2020,2,2)
diff = day1 - day2
prinit(diff.days)
prinit(diff.seconds)
```

### UTC+0 
```python=
dt_now = datetime.now(tz=timezone.utc)
```

### timestamp to datetime
```python=
from datetime import datetime

timestamp = 1545730073
dt_object = datetime.fromtimestamp(timestamp)
```

### format code
> https://docs.python.org/3/library/datetime.html#strftime-and-strptime-format-codes

###  parse 自訂格式
```python!
date_time_str = '2022-12-01 10:27:03.929149'
date_time_obj = datetime.datetime.strptime(date_time_str, '%Y-%m-%d %H:%M:%S.%f')
```

## JSON

### Json String to dict

```python=
qobj = json.loads(jstr)
```

### dict to JSON String
```python=
jstr = json.dumps(dictObj)

# or pretty fromate
jstr = json.dumps(dictObj, indent=4)
```

## escape unescape

### url

```python=

import urllib.parse
逃脫後的文字=urllib.parse.quote(jstr)

原文=urllib.parse.unquote(逃脫後的文字)
```

## 特殊寫法/語法糖

### range()
```python=
for i in range(5):
    print(i)
# out put 0~4

list(range(5))
# [0, 1, 2, 3, 4]
range(1, 11)     # 从 1 开始到 11

range(0, 30, 5)  # 步长为 5
# 0, 5, 10, 15, 20, 25]

range(0, -10, -1) # 负数
# [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
```
> 

## List
用 * 重複多個
```python=
title = ["H"]*3
# ['H','H','H']

names = ["Andy","Peter","Kirin","Tony"]
print(names[-1]) # 取用最後一個 Tony

print(names[1:3]) # 去第1位 第2位 不包含 3  Peter  Kirin

print(names[0:3]) # 去第0位到第2位 不包含 3 Andy Peter  Kirin

print(names[2:]) # 2位 到最後   Kirin Tony

print(names[:]) # 通常用於複製

print(names[::3]) # [0,3,6,9]

numbers = list(range(10))
hiNums = [ 'HI:'+ str(n) for n in numbers ]
print(hiNums)
# ['HI:0', 'HI:1', 'HI:2', 'HI:3', 'HI:4', 'HI:5', 'HI:6', 'HI:7', 'HI:8', 'HI:9']

```

### 加入另一個 List
```python=
list1 = [1, 2]
list2 = [3, 4]
list1.extend(list2)
```

### for取得list index
```python=
for idx,val in enumerate(['a','b','c']):
        print('index of ' + val + ': ' + str(idx))
```

### find by 
```python=
matches = [x for x in lst if fulfills_some_condition(x)]
matches = (x for x in lst if x > 6)
```

### max by
```python=
nst = [ [1001, 0.0009], [1005, 0.07682], [1201, 0.57894], [1677, 0.0977] ]

idx, max_value= max(nst, key=lambda item: item[1])

print('Maximum value:', max_value, "At index:",idx)
```

### sort
```python=
class Score:
    def __init__(self, name, grade, age):
        self.name = name
        self.grade = grade
        self.age = age
    def __repr__(self):
        return repr((self.name, self.grade, self.age))

scores_obj = [
    Score('Jane', 'B', 12),
    Score('John', 'A', 15),
    Score('Dave', 'B', 11)]

# 依照 age 排序
print(sorted(scores_obj, key = lambda s: s.age))
```
output :
```shell=
[('Dave', 'B', 11), ('Jane', 'B', 12), ('John', 'A', 15)]
```


## Class Type

### 取得物件的 Class / Type

```python=
print(type('string'))
# <class 'str'>

print(type(100))
# <class 'int'>

print(type([0, 1, 2]))
# <class 'list'>
```

### Type Callable
```python=
def feeder(get_next_item: Callable[[], str]) -> None:
    ...  # Body
```

## File

### 取得目前腳本當前path
```python=
    wd_path = os.path.dirname(__file__)
    filepath = wd_path + '/two_from_query.sql'

```

### 取得目錄下的檔案列表
```python=
f = []
for (dirpath, dirnames, filenames) in walk(mypath):
    f.extend(filenames)
    break
```
OR
```python=
from os import walk

filenames = next(walk(mypath), (None, None, []))[2]  # [] if no file
```
跨平台Path寫法
```python=
    path = os.path.join(os.path.dirname(__file__), 'folder1','folder2')
    folder = pathlib.Path(path)
    files = [f for f in folder.glob(f"*.csv")]
    print(files)
    return [p.stem for p in files]
```


### read file all content
```python=
f1 = open('file_1.txt','r')
print(f1.read())
f1.close()
# using contenx aoto call file close
with open('file_1.txt','r') as f:
    print(f.read())
```

### save file
```python=
with open('Failed.py', 'w') as file:
    file.write('whatever')
    
file = open('myfile.dat', 'w+') # not exist auto create    
```

## Try Exception

```python=
try:
    with open(filepath,'rb') as f:
        con.storbinary('STOR '+ filepath, f)
    logger.info('File successfully uploaded to '+ FTPADDR)
except Exception as e: # work on python 3.x
    logger.error('Failed to upload to ftp: '+ str(e))
```


## 泛型

### 類別泛型
```python=
T = TypeVar('T')
class DaoUtil(Generic[T]):
```

### 方法泛型

```python=
T = TypeVar('T', int, float, complex)
Vec = Iterable[tuple[T, T]]

def inproduct(v: Vec[T]) -> T: # Same as Iterable[tuple[T, T]]
    return sum(x*y for x, y in v)
```

### 指定泛型是否屬於某個type

```python=
from typing import TypeVar

class Comparable(metaclass=ABCMeta):
    @abstractmethod
    def __lt__(self, other: Any) -> bool: ...
    ... # __gt__ etc. as well

CT = TypeVar('CT', bound=Comparable)

def min(x: CT, y: CT) -> CT:
    if x < y:
        return x
    else:
        return y

min(1, 2) # ok, return type int
min('x', 'y') # ok, return type str
```


## 映射 Reflection

### 型別確認
```python=
    if not isinstance(inv, dict):
        raise TypeError(str(inv) + ' is not dict')
```

### 判斷是否有該屬性
```python=
class Coordinate:
    x = 10
    y = -5
 
point1 = Coordinate() 
print(hasattr(point1, 'x'))

```

### 動態讀取python 檔案的發法

```python=
    wd_path = os.path.dirname(__file__)
    spec = importlib.util.spec_from_file_location("action", f"{wd_path}/{PayloadReqKey.name.get_val(payload)}.py")
    foo = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(foo)
    out_dict: dict = comm_utils.to_dict(foo.run(payload))
    return Response(json.dumps(out_dict), mimetype='application/json')
```

## emum


### Basic Enum

```python=
from enum import Enum

class Color(Enum):
    RED = 1
    GREEN = 2
    BLUE = 3

# Accessing members
print(Color.RED)  # Output: Color.RED
print(Color.RED.name)  # Output: 'RED'
print(Color.RED.value)  # Output: 1

# Iterating through members
for color in Color:
    print(color)
```

### String Enum (str Enum)
```python=
from enum import Enum

class Status(str, Enum):
    PENDING = "pending"
    APPROVED = "approved"
    REJECTED = "rejected"

# Accessing members
print(Status.PENDING)  # Output: Status.PENDING
print(Status.PENDING.name)  # Output: 'PENDING'
print(Status.PENDING.value)  # Output: 'pending'

# Comparing with strings
print(Status.PENDING == "pending")  # Output: True
print(Status.PENDING is "pending")  # Output: False (different identity)

```

###### tags: `python`
