---
title: Pytest 小技巧
tags: [python, pytest]

---

# Pytest 小技巧

## @pytest.fixture()

測試方法前需要呼叫的init method

```python=
import pytest


@pytest.fixture()
def login():
    print("登入")
    return 8


def test_case1():
    print("\n開始執行測試用例1")
    assert 1 + 1 == 2


def test_case2(login):
    print("\n開始執行測試用例2")
    assert 2 + login == 10

```

> test_case2 因參數有 'login' 所以會呼叫 @pytest.fixture() login 方法,並寫入回傳值

### pytest.fixture 的終結函式

```python=
import pytest


@pytest.fixture()
def login(request):
    print("登入" )
    def demo_finalizer():
        print("退出登入")
    # 註冊demo_finalizer為終結函式
    request.addfinalizer(demo_finalizer)
    return 8


def test_case2(login):
    print("\n開始執行測試用例2")
    assert 2 + login == 10

```

> 執行 => `登入` -> `n開始執行測試用例2` -> `退出登入`

> `request` 這個名稱是 pytest 固定的


### pytest.fixture 測試多筆參數

```python=
import pytest

test_key_value_list = [
    {'key': 'k1',
     'val': 'v1'},
    {'key': 'k2',
     'val': 'v2'},
    {'key': 'k',
     'val': 'v'}
]


@pytest.fixture(params=test_key_value_list)
def login(request):
    print("登入" + str(request.param))

    def demo_finalizer():
        print("退出登入")

    # 註冊demo_finalizer為終結函式
    request.addfinalizer(demo_finalizer)
    return request.param

def test_case2(login):
    print("\n開始執行測試用例2")
    print(login)
    assert 2 + 8 == 10


```

執行結果如下

```bash=
tests/test_module.py::test_case2[login0] 登入{'key': 'k1', 'val': 'v1'}
PASSED                          [ 57%]
開始執行測試用例2
{'key': 'k1', 'val': 'v1'}
退出登入

tests/test_module.py::test_case2[login1] 登入{'key': 'k2', 'val': 'v2'}
PASSED                          [ 71%]
開始執行測試用例2
{'key': 'k2', 'val': 'v2'}
退出登入

tests/test_module.py::test_case2[login2] 登入{'key': 'k', 'val': 'v'}
PASSED                          [ 85%]
開始執行測試用例2
{'key': 'k', 'val': 'v'}
退出登入
```

## 測試前的預備進入點

```python=
def setup_module(module):
    app = create_flask_app()
    ctx = app.app_context()
    ctx.push()
    g.privilege_and_policy = {
        '*': {'policy_list': [
            {'all_site': True, 'all_system': True, 'all_station': True, 'all_vip_name': True, 'all_client_id': True,
             'all_origin_port': True, 'all_shipment_mode': True, 'all_destination_port': True}]},
        'shipment_mgmt.shipments.query': {}
    }
    print("-------------- setup before module --------------")
```


###### tags: `python` `pytest`