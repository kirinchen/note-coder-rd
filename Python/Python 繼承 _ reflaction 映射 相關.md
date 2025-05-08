---
title: Python 繼承 / reflaction 映射 相關
tags: [python]

---

# Python Type / 繼承 / reflaction 映射 相關

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


## 類似Java Class 用法
```python
def get_runner_clazz() -> Type[FuncRunner]:
    """
    工廠函式：回傳 FuncRunner 子類別的類別物件
    呼叫者可以這樣使用：
        runner_cls = get_runner()
        runner: FuncRunner = runner_cls(**kwargs)
    """
    return YoutubeTranscriptRunner
```

## 動態加載python script
```python
def inject_all_runner_clazz():  
    if len(_func_runner_clazz_map) > 0:  
        return  
    filenames: List[str] = next(os.walk(wd_path), (None, None, []))[2]  
    for filename in filenames:  
        path = f"{wd_path}/{filename}"  
        name_without_ext, _ = os.path.splitext(filename)  
        lambda_utils.print_log(f"path={path} load...")  
        spec = importlib.util.spec_from_file_location("action", path)  
        foo = importlib.util.module_from_spec(spec)  
        spec.loader.exec_module(foo)  
        if not hasattr(foo, "get_runner_clazz"):  
            continue  
        runner_clazz: Type[FuncRunner] = foo.get_runner_clazz()  
        _func_runner_clazz_map[name_without_ext] = runner_clazz
```
###### tags: `python`