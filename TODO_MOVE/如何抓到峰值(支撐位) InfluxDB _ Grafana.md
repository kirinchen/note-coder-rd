---
title: 如何抓到峰值(支撐位) InfluxDB / Grafana
tags: [influx, grafana]

---

# 如何抓到峰值(支撐位) InfluxDB / Grafana

## 先畫出基本線圖

```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "daily" and
    r.valmean == "high" and
    r.symbol == "${symbolSel}"
  )
```

![](https://i.imgur.com/wZMQlqn.png)

## 找出低點支撐位

![](https://i.imgur.com/BsYZxwu.png)

這時候就可以透過 min來找出一段區間的 最低位

```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "daily" and
    r.valmean == "high" and
    r.symbol == "${symbolSel}"
  )
  |> window(every: 15d)
  |> min()   

```

> 可以去調整 **every: 15d** 來去決定低點的區間 


## 把這些低點變成曲線

![](https://i.imgur.com/SwjDNzu.png)
目前序列都變成單獨的點 可以透過以下語法再把點變成序列

```javascript=
  |> window(every: inf)
```
![](https://i.imgur.com/0i5wyfE.png)

## 整合

![](https://i.imgur.com/q1LR1jM.png)

完整flux
```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "daily" and
    r.valmean == "high" and
    r.symbol == "${symbolSel}"
  )
  |> window(every: 15d)
  |> min()   
  |> window(every: inf)
```

###### tags: `influx` `grafana`