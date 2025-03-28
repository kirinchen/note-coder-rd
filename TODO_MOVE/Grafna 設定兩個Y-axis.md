---
title: Grafna 設定兩個Y-axis
tags: [grafana, 量化交易, linux]

---

常常會遇到兩個不同維度,不同上下界要做比對,結果線圖就被壓縮的很淒慘阿
![](https://hackmd.io/_uploads/BJoGCxEin.png)
如上圖所示,上面的序列被壓縮變成一條線

這時候我們可以設定第2個Y軸來解決

# 解決方式

![](https://hackmd.io/_uploads/SJW0RlNih.png)

在 Edit > Panel > Series overrides > Add Series overrides
加入要分開的序列,並在 Y-axis 選擇 "2"

# 效果如下

![](https://hackmd.io/_uploads/SJNP1WNon.png)

Enjoy!!!