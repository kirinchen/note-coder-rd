---
title: Python 加解密相關
tags: [python, NoPOST]

---

# Python 加解密相關


## sh256 string to string 
```python=
hs = hashlib.sha256(get_some_string().encode('utf-8')).hexdigest()
```

###### tags: `python` `NoPOST`