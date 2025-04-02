---
title: InfluxDB Flux 常用 functions
tags: [influx, grafana]

---

# InfluxDB Flux 常用 functions

## 基本查詢和序列操作
```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "${symbolSel}" and
    r._field == "volume_24h"
  )
```

### 兩個序列 join

```javascript=
join(tables: {t1: t1, t2: t2}, on: ["_time"])
    |> map(fn: (r) => ({ r with _value: (r._value_t1 - r._value_t2)/r._value_t2     }))
    |>map(fn: (r) => ({ r with alias: "120d 投資比" }))     
```

### pivot 同序列不同欄位 join

```javascript=
  |> pivot(rowKey:["_time"], columnKey: ["valmean"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: r.volume / r.open }))    
```


### union 聯集

```javascript=
union(tables: [t1, t2])
```

#### 透過 union 增加 fields 
```javascript=
import "influxdata/influxdb/v1"
import "array"
extend_fields = array.from(rows: [{_value: "test"}]) 
exist_fields=  v1.tagValues(
    bucket: v.bucket,
    tag: "_field",
    predicate: (r) => r._measurement == "mark" and r._field != "h1sdQ2" and r._field != "m5sdQ2" ,
    start: -1y
)
union(tables: [extend_fields, exist_fields])

```


### 今天到現在 range
```javascript=
import "date"
from(bucket: "quote")
  |> range(start: duration(v: string(v: -date.hour(t: now())) +"h")   )
```

### 取得最後15筆
```javascript=
  |> sort(columns: [ "_time"],desc: true)
  |> limit(n: 15)  
```

### duplicate 欄位
```javascript=
	|> duplicate(column: "tag", as: "tag_dup")
```



## 移動平均 / window 相關

### By 個數
```javascript=
  |> movingAverage(n: 2 )
```
### By 時間
```javascript=
    |> timedMovingAverage(every: 1y, period: 5y)
```

### 每一小平均

```javascript=
from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
   |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "ETH" and
    r._field == "price"
  )
  |> window(every: 1h)
  |> mean()   
  |> duplicate(column: "_stop", as: "_time")
  |> window(every: inf)
```

## Time

### 時間粗化
```javascript=
  import "date"
  ...
  |> map(fn: (r) => ({ r with _time: date.truncate(t: r._time, unit: 1d)    }))
```

### 一定時間內的Min / Max
```javascript=
t1 = from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r._field == "price" and
    r.symbol == "${symbolSel}" 
  )
  |> window(every: 1h)
  |> max()   
  |> window(every: inf)
  |> map(fn: (r) => ({ r with alias:"1h max" }))

t1
```

### 時間shift
```javascript=
  |> timeShift(duration: 120d)
```

### 時間增加...用在range()
```javascript=
maShiftDuration= duration(v: "-"+string(v: (5*${y1_ma_n})) +"m")

ser= from(bucket: "quote")
  |> range(start: date.add(d: maShiftDuration , to : v.timeRangeStart  ) , stop:v.timeRangeStop)
...
```

## value function / map 常用
### IF ... Else...
```javascript=
 |>map(fn: (r) => ({ r with alias: if r._field == "quote" then "Ordered-"+r.side else "StartOrder-"+r.side}))   

```

### Math.abs
```javascript=
import "math"
math.abs(x: -1.22)
```

### findColumn Example
```javascript=
a=  from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
   |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "${symbol}" and
    r._field == "price"
  )
  |>max()
  |> findColumn(
      fn: (key) => key._field == "price",
      column: "_value"
    )


from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop:v.timeRangeStop)
   |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "${symbol}" and
    r._field == "price"
  )
  |>map(fn: (r) => ({ r with _value: a[0] }))   
```



## string

### string replace


```javascript=
import "strings"
...
  |> movingAverage(n: int( v: strings.replaceAll(v: "${period}", t: "m", u: ""))/5 )
```

## object to stream
```javascript=
import "experimental"
import "array"
import "experimental/record"
user = {firstName: "John", lastName: "Doe", age: "42"}

dKeys= experimental.objectKeys(o: user)// Returns [firstName, lastName, age]
showKeys = dKeys |> array.map(fn: (x) => ({_value: x}))
showKV =  dKeys |> array.map(fn: (x) => ({field: x , val : record.get(r: user, key: x, default: "") }))

array.from(rows: showKV )
```

###### tags: `influx` `grafana`