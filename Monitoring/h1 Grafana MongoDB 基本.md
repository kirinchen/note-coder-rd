---
title: h1 Grafana MongoDB 基本
tags: [grafana, 量化交易]

---

# Grafana MongoDB 基本操作

## Query 
```javascript=
bzklog.order_model.aggregate([
    { $match : { exchange : "binance_dev" , type:"LIMIT" , pair:"BTC"  } },
    { $sort : { createAt : 1 } },
    {
        $project: {
            _id:1,
            "price":"$orig.price",
            "exchange":"$exchange",
            "_time":"$createAt"
        }
    }
]
)
```

