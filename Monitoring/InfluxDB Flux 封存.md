---
title: InfluxDB Flux 封存
tags: [influx]

---

# InfluxDB Flux 封存

## MACD DIF

### DIF_Slow

```javascript=
import "strings"
from(bucket: "quote")
  |> range(start: 2022-03-10T09:08:50Z, stop:2022-03-11T09:08:50Z)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "BTC" 
  )
  |> timeShift(duration: 5m)  
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: (r.low + r.open +r.close + r.high)/4.0 }))  
  |> map(fn: (r) => ({ r with cp: r._value   }))
  |> movingAverage(n: int( v: strings.replaceAll(v: "600m", t: "m", u: ""))/5 )
  |> map(fn: (r) => ({ r with _value: (r.cp-r._value)   }))

  |>map(fn: (r) => ({ r with alias: "DIF_Slow" })) 
```

### DIF
```javascript=
import "strings"
from(bucket: "quote")
  |> range(start: 2022-03-10T09:08:50Z, stop:2022-03-11T09:08:50Z)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "BTC" 
  )
  |> timeShift(duration: 5m)  
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: (r.low + r.open +r.close + r.high)/4.0 }))  
  |> map(fn: (r) => ({ r with cp: r._value   }))
  |> movingAverage(n: int( v: strings.replaceAll(v: "60m", t: "m", u: ""))/5 )
  |> map(fn: (r) => ({ r with _value: (r.cp-r._value)   }))
```

### macd

```javascript=
import "strings"
from(bucket: "quote")
  |> range(start: 2022-03-10T09:08:50Z, stop:2022-03-11T09:08:50Z)
  |> filter(fn: (r) =>
    r._measurement == "realtime" and
    r.symbol == "BTC" 
  )
  |> timeShift(duration: 5m)  
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with _value: (r.low + r.open +r.close + r.high)/4.0 }))  
  |> map(fn: (r) => ({ r with cp: r._value   }))
  |> movingAverage(n: int( v: strings.replaceAll(v: "60m", t: "m", u: ""))/5 )
  |> map(fn: (r) => ({ r with _fast: r.cp -r._value    }))
  |> map(fn: (r) => ({ r with _value: r.cp   }))
  |> movingAverage(n: int( v: strings.replaceAll(v: "600m", t: "m", u: ""))/5 )
  |> map(fn: (r) => ({ r with _slow: r.cp -r._value   }))
  |> map(fn: (r) => ({ r with _value: r._slow - r._fast   }))
```


## K 壓力
```javascript=
import "math"
sr = from(bucket: "quote")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "candlestick")
  |> filter(fn: (r) => r["interval"] == "5m")
  |> filter(fn: (r) => r["_field"] == "open" or r["_field"] == "low" or r["_field"] == "high" or r["_field"] == "close")
  |> filter(fn: (r) => r["symbol"] == "BTC")
  |> timeShift(duration: 5m)  

//   
// sr
//   |>map(fn: (r) => ({ r with alias: r._field }))   
//   |> yield(name: "fields") 

sr 
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> map(fn: (r) => ({ r with bottom: if r.open < r.close then r.open else r.close , top: if r.open > r.close then r.open else r.close }))   
  |> map(fn: (r) => ({ r with top_dif: r.high - r.top  , bottom_dif: r.low - r.bottom , solid_width: r.top - r.bottom }))    
  |> map(fn: (r) => ({ r with pressure: if r.solid_width>0 then (r.top_dif + r.bottom_dif)/r.solid_width else 0.0 }))    
  |> map(fn: (r) => ({ r with _value: r.pressure })) 



 



```

###### tags: `influx`