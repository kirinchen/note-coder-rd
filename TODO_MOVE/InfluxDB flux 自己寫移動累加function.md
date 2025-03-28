---
title: InfluxDB flux 自己寫移動累加function
tags: [influx]

---

# InfluxDB flux 自己寫移動累加function

例如:我要得知 6小時交易量 / 6小時累積雨量 等等 如何Query

不幸的 flux 內建只有移動平均

* [movingAverage() function](https://docs.influxdata.com/influxdb/cloud/reference/flux/stdlib/built-in/transformations/movingaverage/)
* [timedMovingAverage() function]([https://docs.influxdata.com/influxdb/cloud/reference/flux/stdlib/built-in/transformations/movingaverage/](https://docs.influxdata.com/influxdb/cloud/reference/flux/stdlib/built-in/transformations/timedmovingaverage/))

可以使用

## 那就自己寫一個

這時候就自己寫一個function
直接看語法:

```javascript=
timedMovingSum = (every, period, column="_value", tables=<-) =>
  tables
    |> window(every: every, period: period)
    |> sum(column:column)
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)
```

## usage

```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "${symbolSel}" and
    r._field == "${col}"
  )
  |> difference()
  |> difference()
  |>timedMovingSum(every: 5m, period: 1h)
  |> movingAverage(n: int(v: strings.replaceAll(v: "${period}", t: "h", u: "")  )*12  )
  |>map(fn: (r) => ({ r with alias: "${period}" }))   
```

### Parameters

#### every
The frequency of time windows.

Data type: Duration

#### period
The length of each averaged time window. A negative duration indicates start and stop boundaries are reversed.

Data type: Duration

#### column
The column used to compute the moving sum. Defaults to "_value".

Data type: String


###### tags: `influx`