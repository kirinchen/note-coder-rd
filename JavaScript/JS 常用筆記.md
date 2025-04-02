---
title: JS 常用筆記
tags: [javascript]

---

# JS 常用筆記

## Date 
### plus by senconds
```javascript=
var end = new Date();
var endiso = end.toISOString();
end.setSeconds(end.getSeconds() - 5*60);
var startiso = end.toISOString();
console.log(startiso);
console.log(endiso);
```

### parse ISO8601
```javascript=
new Date(iso_8601_dateString)
```

### plus by days
```javascript=
  var dat = new Date(this.valueOf()); // (1)
  dat.setDate(dat.getDate() + days);  // (2)
```

### data to iso8601
```javascript=
let today = new Date('05 October 2011 14:48 UTC')
console.log(today.toISOString())  // Returns 2011-10-05T14:48:00.000Z
```

### 比較先後
```javascript=
const lastOdAt ="2021-09-01T09:50:36.017000+00:00";
const lastD = new Date(lastOdAt);
const now = new Date();

console.log(lastD<now)
```

### check 是 vaild 的Date
```javascript=
const lastOdAt ="null";
const lastD = new Date(lastOdAt);
console.log(isNaN(lastD))
```

## Number

### MIN MAX Value
```
Number.MAX_VALUE 
Number.MIN_VALUE 

Number.MIN_SAFE_INTEGER

```

## Array

### for...of
```javascript=
const array1 = ['a', 'b', 'c'];

for (const element of array1) {
  console.log(element);
}

// expected output: "a"
// expected output: "b"
// expected output: "c"
```

### add other array
```javascript=
const array1 = ['a', 'b', 'c'];
const array2 = ['d', 'e', 'f'];
const array3 = array1.concat(array2);
console.log(array3);
// expected output: Array ["a", "b", "c", "d", "e", "f"]
```

### add new one

```javascript=
arr.push("Hola");
```

### find
```javascript=
const array1 = [5, 12, 8, 130, 44];
const found = array1.find(element => element > 10);
console.log(found);
// Expected output: 12
```


### 判斷陣列是否包含特定的元素

```javascript=
const array1 = [1, 2, 3];

console.log(array1.includes(2));
// expected output: true

const pets = ['cat', 'dog', 'bat'];

console.log(pets.includes('cat'));
// expected output: true

console.log(pets.includes('at'));
```


## JSON

### JSON.stringify()

```javascript=
console.log(JSON.stringify({ x: 5, y: 6 }));
// expected output: "{"x":5,"y":6}"

console.log(JSON.stringify([new Number(3), new String('false'), new Boolean(false)]));
// expected output: "[3,"false",false]"
```

## object

### check empty

```javascript=
function isEmpty(obj) {
    return Object.keys(obj).length === 0;
}
```

### foreach key value

```javascript=
const test = {a: 1, b: 2, c: 3};

for (const [key, value] of Object.entries(test)) {
  console.log(key, value);
}

```

### merge
```javascript=
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };
var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, 目標物件本身也被改變。
```

### clone
```javascript
Object.assign({},o1)
```

### 
```javascript=
o = new Object();
o.prop = 'exists';

o.hasOwnProperty('prop');   // 回傳 true
```


### 測試屬性是否存在
```javascript=
o = new Object();
o.prop = 'exists';

function changeO() {
  o.newprop = o.prop;
  delete o.prop;
}

o.hasOwnProperty('prop');   // 回傳 true
changeO();
o.hasOwnProperty('prop');   // 回傳 false
```

###### tags: `javascript`