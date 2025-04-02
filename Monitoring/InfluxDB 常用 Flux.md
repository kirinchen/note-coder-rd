---
title: InfluxDB 常用 Flux
tags: [influx]

---

# InfluxDB 常用 Flux

## 交易量標準差Rate
```flux
movingStddev = (every, period, column="_value", tables=<-) =>
  tables
    |> window(every: every, period: period)
    |> stddev(column:column)
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)


ser = from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "realtime")
  |> filter(fn: (r) => r["_field"] == "quoteAssetVolume")
  |> filter(fn: (r) => r["symbol"] == "BTC")

stds= ser
|> movingStddev(every:5m, period: ${period})

avgs= ser
|> timedMovingAverage(every:5m, period: ${period2})

join(tables: {stds: stds, avgs: avgs}, on: ["_time"])
    |> map(fn: (r) => ({ r with _value: r._value_stds/r._value_avgs    }))
    |>map(fn: (r) => ({ r with alias: "交易量stddev_rate" }))   
```

## 兩個 window Aggregates 如何做計算
```javascript=
import "math"

moveSum = (every, period, column="_value", tables=<-) =>
  tables
    |> window(every: every, period: period)
    |> sum(column:column)
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)

a = from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
   |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "BTC" 
  )
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: (r.low + r.open +r.close + r.high)/4.0 }))  
  |> map(fn: (r) => ({ r with orgv: r._value }))  
  |> difference()

positive = a
    |>map(fn: (r) => ({ r with _value: if r._value > 0 then r._value else 0.0 }))   
    |>moveSum(every: 5m, period: 30m)
    |> set(key: "tag", value: "positive")
    |> group()
    
negative = a
    |>map(fn: (r) => ({ r with _value: if r._value < 0 then r._value else 0.0 }))   
    |>moveSum(every: 5m, period: 30m)
    |> set(key: "tag", value: "negative")
    |> group()

union(tables: [positive, negative])
  |> pivot(rowKey:["_time"], columnKey: ["tag"], valueColumn: "_value")
  |>map(fn: (r) => ({ r with _value: if r.positive > math.abs(x: r.negative)  then r.positive else r.negative }))   
```

## 布林通道

```javascript=

sr= from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
   |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "${symbol}" 
  )
  |> timeShift(duration: 5m)  
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: (r.low + r.close + r.high)/3.0 }))  

_sd=   sr
    |> window(period: ${period},every: 5m)
    |> stddev()
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)
 

_ma=   sr
    |> window(period: ${period},every: 5m)
    |> mean()
    |> duplicate(column: "_stop", as: "_time")
    |> window(every: inf)

_ma
    |>map(fn: (r) => ({ r with alias: "ma_${period}" }))   
    |> yield(name: "ma")


join(tables: {_sd: _sd, _ma: _ma}, on: ["_time"])    
    |>map(fn: (r) => ({ r with _value: r._value__ma+(2.0 * r._value__sd) , alias : "BOL/TOP" }))   
    |> yield(name: "top")

join(tables: {_sd: _sd, _ma: _ma}, on: ["_time"])    
    |>map(fn: (r) => ({ r with _value: r._value__ma-(2.0 * r._value__sd) , alias : "BOL/BOTTOM" }))    
    |> yield(name: "bottom")   

```

## 1k OCHL 
```javascript=
timeWin = (every, fn, field, tables=<-) =>
  tables
  |> filter(fn: (r) =>  r["_field"] == field )
  |> aggregateWindow(every: every, fn: fn)

ser= from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "candlestick")
  |> filter(fn: (r) => r["_field"] == "close" or r["_field"] == "high" or r["_field"] == "low" or r["_field"] == "open")
  |> filter(fn: (r) => r["symbol"] == "BTC")
  |> filter(fn: (r) => r["interval"] == "5m")


h1Open=ser
  |> timeWin(every: 1h, field: "open", fn: first)

h1Close=ser
  |> timeWin(every: 1h, field: "close", fn: last)

h1Low=ser
  |> timeWin(every: 1h, field: "low", fn: min)


h1High=ser
  |> timeWin(every: 1h, field: "high", fn: max)



h1Set= union(tables: [h1Low, h1High,h1Open,h1Close])
  |> yield(name: "h1HighLow")
```

###### tags: `influx`