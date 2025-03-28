---
title: Python string 全攻略
tags: [python]

---

# Python string 全攻略

## str.format()

```python=

# older
s = 'I am %s %s. %s'

s.format('Monkey', 'D','Luffy')

# newer

s = 'I am {first_name} {middle_name}. {last_name}'

s.format(first_name='Monkey', middle_name='D', last_name='Luffy')

```

### 如何 escape { }

```ㄣ
>>> x = " {{ Hello }} {0} "
>>> print(x.format(42))
' { Hello } 42 '
```

###### tags: `python`