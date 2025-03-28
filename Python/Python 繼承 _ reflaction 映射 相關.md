---
title: Python 繼承 / reflaction 映射 相關
tags: [python]

---

# Python 繼承 / reflaction 映射 相關

## 建構子

```python=
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```

## Inheriting from a generic abstract class

```python=
from abc import ABC, abstractmethod
from typing import Dict, Generic, List, TypeVar

T = TypeVar("T")


class FooGenericAbstract(ABC, Generic[T]):

    @abstractmethod
    def func(self) -> T:
        pass


class Foo(FooGenericAbstract[Dict[str, int]]): 

    def func(self) -> Dict[str, str]:
        pass
```

## 取得所有的 annotations
```python=
def list_annotations_with_base(cls: type) -> Dict[str, type]:
    ans: Dict[str, type] = dict()
    for b_cls in cls.__bases__:
        ans.update(list_annotations_with_base(b_cls))
    ans.update(cls.__annotations__)
    return ans
```

###### tags: `python`