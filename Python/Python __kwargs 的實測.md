---
title: Python **kwargs 的實測
tags: [python]

---

# Python **kwargs 的實測


## 都必填

### 都有待入值

```python=
def test_func(a1, a2):
    print(a1)
    print(a2)


if __name__ == '__main__':
    d = dict()
    d['a1'] = 1
    d['a2'] = 1

    test_func(**d)
```

out put
```bash=
1
2
```

### 有值未帶入
```python=
def test_func(a1, a2):
    print(a1)
    print(a2)


if __name__ == '__main__':
    d = dict()
    d['a1'] = 1

    test_func(**d)
```

out put
```bash=
Traceback (most recent call last):
  File "xxx/main.py", line 44, in <module>
    test_func(**d)
TypeError: test_func() missing 1 required positional argument: 'a2'
```


## 有預設值參數


```python=
def test_func(a1, a2=None):
    print(a1)
    print(a2)


if __name__ == '__main__':
    d = dict()
    d['a1'] = 1

    test_func(**d)
```


output
```bash=
1
None
```

## 帶入沒有該參數的key & value (unexpected-keyword-arguments)

```python=
def test_func(a1, a2=None):
    print(a1)
    print(a2)


if __name__ == '__main__':
    d = dict()
    d['a1'] = 1
    d['a2'] = 2
    d['a3'] = 3

    test_func(**d)
```


output
```bash=
Traceback (most recent call last):
  File "xxx/main.py", line 46, in <module>
    test_func(**d)
TypeError: test_func() got an unexpected keyword argument 'a3'
```

## 解決unexpected-keyword-arguments 的作法

> 適用於向下相容


```python=
def test_func(a1, a2=None,**kwargs):
    print(a1)
    print(a2)
    print(kwargs['a3'])


if __name__ == '__main__':
    d = dict()
    d['a1'] = 1
    d['a2'] = 2
    d['a3'] = 3

    test_func(**d)
```


output
```bash=
1
2
3
```


###### tags: `python`