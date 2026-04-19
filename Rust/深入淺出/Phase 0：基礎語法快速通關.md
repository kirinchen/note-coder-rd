# Phase 0：基礎語法快速通關

  

> 目標：掌握 Rust 的書寫規範，理解它如何處理最基本的資料。

  

---

  

## 1. 基礎型別（Data Types）

  

Rust 是**靜態型別語言**，每個變數在編譯期就必須知道它的型別。

大多數時候，Rust 可以自動推斷型別，但你也可以明確標註。

  

### 1.1 Scalar 純量型別

  

純量代表「單一個值」，共四種：

  

#### 整數（Integer）

  

```rust

let a: i8  = 127;          // 有號 8-bit:  -128 ~ 127

let b: u8  = 255;          // 無號 8-bit:    0 ~ 255

let c: i32 = -1_000_000;   // 有號 32-bit（最常用，預設）

let d: u64 = 18_446_744;   // 無號 64-bit

let e: isize = 100;        // 平台相關大小（64-bit 系統 = i64）

let f: usize = 100;        // 常用於陣列索引、長度

```

  

> **與 Java 的差異：** Java 的 `int` 固定是 32-bit，Rust 的 `i32` 也是，但 Rust 多了

> `u8`～`u128` 的無號型別，以及 `isize`/`usize` 這兩個平台相關型別。

  

> **底線分隔符：** `1_000_000` 和 `1000000` 完全相同，底線只是視覺分隔用。

  

#### 浮點數（Float）

  

```rust

let x: f32 = 3.14;    // 單精度（較少用）

let y: f64 = 3.14;    // 雙精度（預設，精度更高）

  

// 注意：不能混合型別做運算！

// let z = x + y;  // ❌ 編譯錯誤：mismatched types

let z = x as f64 + y;  // ✅ 必須顯式轉型

```

  

#### 布林（Boolean）

  

```rust

let t: bool = true;

let f: bool = false;

// Rust 不允許用整數代替布林，不像 C

// if 1 { }  // ❌ 錯誤

```

  

#### 字元（Char）

  

```rust

let c: char = 'A';

let emoji: char = '😀';   // ✅ Rust 的 char 是 4 bytes Unicode，可存任何 Unicode 字元

let chinese: char = '中';  // ✅ 完全合法

  

// 注意：char 用單引號 ' '，字串用雙引號 " "

```

  

> **與 C 的最大差異：** C 的 `char` 是 1 byte（ASCII），Rust 的 `char` 是 4 bytes（Unicode Scalar Value）。

  

---

  

### 1.2 Compound 複合型別

  

複合型別可以把多個值組合成一個型別。

  

#### 元組（Tuple）— 固定長度、可混合型別

  

```rust

let tup: (i32, f64, char) = (42, 3.14, 'A');

  

// 解構（Destructuring）

let (x, y, z) = tup;

println!("x={}, y={}, z={}", x, y, z);

  

// 用索引存取

println!("第一個元素: {}", tup.0);

println!("第二個元素: {}", tup.1);

  

// 空元組 () 叫做 unit，代表「沒有值」（類似 void）

let nothing: () = ();

```

  

#### 陣列（Array）— 固定長度、相同型別（存在 Stack 上）

  

```rust

let arr: [i32; 5] = [1, 2, 3, 4, 5];  // 型別標註：[元素型別; 長度]

let zeros = [0; 10];                   // 建立 10 個 0 的陣列

  

println!("第一個: {}", arr[0]);

println!("長度: {}", arr.len());

  

// 越界存取會在 runtime panic，不會像 C 那樣造成 undefined behavior

// let invalid = arr[10];  // ❌ 會 panic: index out of bounds

```

  

> **Array vs Vec：** Array 長度固定，存在 Stack；Vec 長度可變，存在 Heap。

> 大多數時候你會用 `Vec<T>`，但 Array 在效能敏感場合有優勢。

  

---

  

## 2. 函式（Functions）

  

### 2.1 基本語法

  

```rust

// 命名慣例：snake_case（不是 camelCase）

fn say_hello(name: &str, times: u32) {

    for i in 0..times {

        println!("Hello, {}! (第 {} 次)", name, i + 1);

    }

}

  

// 有回傳值：用 -> 指定型別

fn add(x: i32, y: i32) -> i32 {

    x + y  // ⚠️ 注意：沒有分號！這是「表達式」，代表回傳這個值

}

```

  

### 2.2 語句（Statement）vs 表達式（Expression）— ⚠️ Rust 最重要的概念之一

  

這是 Rust 最讓新手困惑的地方，必須完全理解：

  

```rust

fn demonstrate() {

    // 語句（Statement）：執行動作，不回傳值，結尾有分號

    let x = 5;           // 這是語句

    println!("{}", x);   // 這也是語句（macro 呼叫）

  

    // 表達式（Expression）：計算並回傳一個值，沒有分號

    let y = {

        let inner = 3;

        inner * 2        // ← 沒有分號 = 這個 block 的回傳值是 6

    };

    // y == 6

  

    // 函式的隱含 return

    // 以下兩個函式完全等價：

}

  

fn explicit_return(x: i32) -> i32 {

    return x * 2;  // 用 return 關鍵字（通常只在提早返回時用）

}

  

fn implicit_return(x: i32) -> i32 {

    x * 2  // 最後一個表達式就是回傳值（慣用寫法）

}

  

// ⚠️ 常見錯誤：加了分號變成 ()

fn wrong_return(x: i32) -> i32 {

    x * 2;  // ❌ 這行加了分號變成「語句」，回傳 () 而不是 i32

            //    編譯器會報錯：expected `i32`, found `()`

}

```

  

> **直覺理解：** 想像分號是「我不在乎這個值，丟掉它」的符號。

> 不加分號 = 「把這個值往上傳」。

  

---

  

## 3. 控制流程（Control Flow）

  

### 3.1 if 表達式

  

```rust

let number = 7;

  

// 基本 if/else

if number > 5 {

    println!("大於 5");

} else if number == 5 {

    println!("等於 5");

} else {

    println!("小於 5");

}

  

// ⚠️ 重點：if 在 Rust 中是「表達式」，可以回傳值！

let description = if number > 5 { "大" } else { "小" };

//  類似 Java/Python 的三元運算子，但更強大且清晰

//  注意：兩個分支的型別必須相同

  

// 可以用在 let 綁定：

let abs_value = if number >= 0 { number } else { -number };

```

  

### 3.2 loop — 無限迴圈

  

```rust

let mut counter = 0;

  

// loop 也是表達式，可以回傳值

let result = loop {

    counter += 1;

    if counter == 10 {

        break counter * 2;  // break 可以帶值，作為 loop 的回傳值

    }

};

println!("result = {}", result);  // 20

  

// 迴圈標籤（Nested loop 的 break/continue 控制）

'outer: loop {

    'inner: loop {

        break 'outer;  // 直接跳出外層迴圈

    }

}

```

  

### 3.3 while — 條件迴圈

  

```rust

let mut n = 3;

while n > 0 {

    println!("{}!", n);

    n -= 1;

}

println!("發射！");

```

  

### 3.4 for + Iterator — 最推薦的迴圈方式 ⭐

  

```rust

// 遍歷集合（最常用）

let fruits = ["apple", "banana", "cherry"];

for fruit in fruits {

    println!("水果: {}", fruit);

}

  

// 數字範圍

for i in 0..5 {    // 0, 1, 2, 3, 4（不包含 5）

    print!("{} ", i);

}

  

for i in 0..=5 {   // 0, 1, 2, 3, 4, 5（包含 5）

    print!("{} ", i);

}

  

// enumerate() 同時取得索引和值

for (index, fruit) in fruits.iter().enumerate() {

    println!("{}: {}", index, fruit);

}

  

// rev() 反向迭代

for i in (0..5).rev() {

    println!("{}", i);  // 4, 3, 2, 1, 0

}

```

  

> **為什麼推薦 for + Iterator？**

> 1. 不會有越界問題（不像 `while i < arr.len()`）

> 2. 效能極佳（編譯器可以優化）

> 3. 表達意圖更清晰

  

---

  

## 4. Macro 巨集初探

  

### 4.1 辨識 Macro

  

Macro 的呼叫以 `!` 結尾，這是它們與普通函式的視覺區別：

  

```rust

println!("Hello, {}!", "world");  // macro

vec![1, 2, 3];                    // macro

panic!("發生嚴重錯誤");             // macro

format!("數值是 {}", 42);          // macro，回傳 String

assert!(1 + 1 == 2);              // macro，測試用

assert_eq!(2 + 2, 4);            // macro，比較兩個值

```

  

### 4.2 為什麼需要 Macro？

  

```rust

// println! 為什麼不是普通函式？

// 因為它接受不定數量、不定型別的參數：

println!("一個參數");

println!("兩個: {} {}", 1, "hello");

println!("三個: {} {} {}", 1, 2.0, 'c');

  

// 普通函式無法做到這件事（在沒有 macro 系統的語言中需要用 varargs 等機制）

// Macro 在「編譯期」展開成對應的程式碼，所以沒有執行期的負擔

```

  

### 4.3 格式化字串

  

```rust

let name = "Kirin";

let score = 95;

  

println!("姓名: {}", name);           // 基本替換

println!("分數: {:>10}", score);      // 靠右對齊，寬度 10

println!("分數: {:<10}", score);      // 靠左對齊

println!("浮點: {:.2}", 3.14159);    // 保留 2 位小數

println!("Debug: {:?}", [1, 2, 3]);  // Debug 輸出

println!("十六進位: {:x}", 255);      // ff

println!("二進位: {:b}", 10);         // 1010

```

  

---

  

## 練習題說明

  

```bash

# 練習 1：溫度轉換器

cargo run -p phase0_basics --example ex01_temperature

  

# 練習 2：費氏數列產生器

cargo run -p phase0_basics --example ex02_fibonacci

  

# 練習 3：猜數字遊戲

cargo run -p phase0_basics --example ex03_guessing_game

  

# 執行所有單元測試

cargo test -p phase0_basics

```

  

## 重點複習

  

| 概念 | Rust | Java/Python 對比 |

|------|------|-----------------|

| 整數預設 | `i32` | Java: `int`, Python: `int`（無限精度） |

| 浮點預設 | `f64` | Java: `double`, Python: `float` |

| char 大小 | 4 bytes (Unicode) | Java: 2 bytes (UTF-16) |

| 變數可變性 | 預設不可變（`let`） | Java/Python 預設可變 |

| 函式回傳 | 最後表達式（無分號） | 必須用 `return` |

| if | 是表達式，可回傳值 | Java 中 if 是語句 |