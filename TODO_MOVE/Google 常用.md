---
title: Google 常用
tags: [javascript, appScript]

---

# Google 常用

## 抓一個範圍取得該idx
```javascript=
function getSomeIdx(wSheet,rng,someOne){
var rows=  wSheet.getRange(rng).getValues();
 for  (var i in rows){
   if(rows[i] == someOne){
     return i;
       }
     }
  }
```

## 抓取某個 cell 該 value
```javascript=
rateSheet.getRange(y,x,height,width).getValue();

// y= 直的idx 1 base 1
// x= 橫的idx 1 base 1
```

## 取得某sheet
```javascript=
var smallRate = SpreadsheetApp.getActive().getSheetByName('SmallRate');
```

###### tags: `appScript` `javascript`