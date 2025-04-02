---
title: Flask 常用小筆記
tags: [python, flask]

---

# Flask 常用小筆記

# 取得payload (JSON)
```python=
payload:dict  = request.json
```

# Response Json

```python=
    d:dict = ans.gen_flat_dict()
    return jsonify(d), 201
```

###### tags: `python` `flask`