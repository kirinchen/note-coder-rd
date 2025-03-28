---
title: Python closure & late binding 陷阱
tags: [python]

---

# Python closure & late binding 陷阱

不囉嗦直接看code

```python=
# Online Python compiler (interpreter) to run Python online.
# Write Python 3 code in this online editor and run it.
class TC:

    def __init__(self,
                 condition_list: list = ['A']):
        self.condition_list = condition_list

    def add(self):
        temp_tc = TC()
        self.condition_list.extend(temp_tc.condition_list)
        
tc1 = TC()
tc1.add()
print(tc1.condition_list)
tc2 = TC()
tc2.add()
print(tc2.condition_list)
tc3 = TC()
tc3.add()
print(tc3.condition_list)
```

## 很直覺的答案應該是如下

```bash=
['A', 'A']
['A', 'A']
['A', 'A']
```

## 但是結果卻是如下
```bash=
['A', 'A']
['A', 'A', 'A', 'A']
['A', 'A', 'A', 'A', 'A', 'A', 'A', 'A']
```

## 結論
很明顯這有很嚴重的 memory leak 的 issue
主要是closure讓區域變數無法release, condition_list 看似已經從內存中消失了，但依然能夠訪問到 condition_list 這個非局部變量！

###### tags: `python`

