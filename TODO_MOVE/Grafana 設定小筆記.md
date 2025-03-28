---
title: Grafana 設定小筆記
tags: [grafana, influx]

---

# Grafana 設定小筆記




## 同時設定不同的序列的顯示方式

這時候需要用命名規則

像是我有以下的序列
* order/price/LONG/LIMIT
* order/price/LONG/STOP_MARKET
* order/price/LONG/TAKE_PROFIT

只需要在
Series overrides > Alias or regex set : **/order/**
![](https://i.imgur.com/k9KbFOI.png)

> 這樣就不用每個都設定了

## 要畫出區間的效果(ex : 布林通道)
![](https://i.imgur.com/1Ys03Vj.png)

1. Series overrides >  Alias or regex set : 選擇要得序列
2. 選擇 **Fill below to ** 
3. 選擇要話到底部的序列
4. DONE

![](https://i.imgur.com/1Zz7sfM.png)

> 紅色箭頭是系統自動填上的

## Grafana 設定序列的title

![](https://i.imgur.com/4l3bP50.png)
在 Field > display name 設定 **${__field.labels.alias}**
並且在該序列增加
```javascript=
 |>map(fn: (r) => ({ r with alias: "顯示名稱" }))     
```

###### tags: `grafana` `influx`