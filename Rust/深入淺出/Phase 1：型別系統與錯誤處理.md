# Phase 1：型別系統與錯誤處理

  

> 目標：轉換思維 — 用 Rust 的方式思考資料結構與錯誤。

> Rust 沒有繼承，沒有 null，沒有 exception，但更強大、更安全。

  

---

  

## 1. Struct（結構體）

  

### 1.1 命名結構體

  

```rust

// 定義資料結構（類似 Java 的 class，但只有資料，沒有行為）

struct User {

    username: String,

    email: String,

    age: u32,

    active: bool,

}

  

// 建立實例

let user = User {

    username: String::from("kirin"),

    email: String::from("kirin@example.com"),

    age: 25,

    active: true,

};

  

// 欄位更新語法（Field Update Syntax）

let user2 = User {

    email: String::from("new@example.com"),

    ..user  // 其他欄位從 user 複製

};

```

  

### 1.2 用 impl 加入行為

  

```rust

impl User {

    // 關聯函式（類似 static method）— 不接受 self

    fn new(username: &str, email: &str) -> Self {

        User {

            username: username.to_string(),

            email: email.to_string(),

            age: 0,

            active: true,

        }

    }

  

    // 方法（接受 &self = 不可變借用）

    fn display(&self) {

        println!("{} <{}>", self.username, self.email);

    }

  

    // 可變方法（接受 &mut self）

    fn deactivate(&mut self) {

        self.active = false;

    }

}

```

  

### 1.3 Tuple Struct

  

```rust

struct Color(u8, u8, u8);    // RGB

struct Point(f64, f64);

  

let red = Color(255, 0, 0);

let origin = Point(0.0, 0.0);

  

println!("紅色: ({}, {}, {})", red.0, red.1, red.2);

```

  

---

  

## 2. Enum（枚舉）— Rust 的超級武器

  

Rust 的 Enum 遠比 Java 的 Enum 強大，因為它的**每個 variant 都可以攜帶不同型別的資料**。

  

```rust

// 基本 Enum

enum Direction {

    North,

    South,

    East,

    West,

}

  

// 攜帶資料的 Enum（這才是精華）

enum Shape {

    Circle(f64),                    // 半徑

    Rectangle(f64, f64),            // 寬、高

    Triangle { base: f64, height: f64 },  // 具名欄位

}

  

impl Shape {

    fn area(&self) -> f64 {

        match self {

            Shape::Circle(r) => std::f64::consts::PI * r * r,

            Shape::Rectangle(w, h) => w * h,

            Shape::Triangle { base, height } => 0.5 * base * height,

        }

    }

}

```

  

---

  

## 3. Pattern Matching（模式比對）— Rust 的靈魂

  

### 3.1 基本 match

  

```rust

let num = 3;

  

match num {

    1 => println!("一"),

    2 | 3 => println!("二或三"),      // | 表示「或」

    4..=6 => println!("四到六"),      // 範圍

    _ => println!("其他"),            // _ 是萬用（必須存在，確保完整覆蓋）

}

  

// match 也是表達式，可以回傳值

let description = match num {

    1 => "one",

    2 => "two",

    3 => "three",

    _ => "many",

};

```

  

### 3.2 解構 Enum

  

```rust

let shape = Shape::Rectangle(4.0, 6.0);

  

match shape {

    Shape::Circle(r) => println!("圓形，半徑 {}", r),

    Shape::Rectangle(w, h) => println!("長方形 {}x{}", w, h),

    Shape::Triangle { base, height } => println!("三角形"),

}

```

  

### 3.3 if let — 只關心一個 variant

  

```rust

// 比 match 更簡潔，當你只關心一種情況時

let some_value = Some(42);

  

if let Some(x) = some_value {

    println!("有值: {}", x);

}

// 等價於：

match some_value {

    Some(x) => println!("有值: {}", x),

    None => {},

}

```

  

---

  

## 4. Option — 取代 null

  

```rust

// Option<T> 的定義（標準庫內建）：

// enum Option<T> {

//     Some(T),

//     None,

// }

  

fn find_user(id: u32) -> Option<String> {

    if id == 1 {

        Some(String::from("Kirin"))

    } else {

        None  // 明確表示「找不到」，而不是回傳 null

    }

}

  

let result = find_user(1);

  

// 方式 1：match（最明確）

match result {

    Some(name) => println!("找到使用者: {}", name),

    None => println!("找不到"),

}

  

// 方式 2：unwrap_or（提供預設值）

let name = find_user(99).unwrap_or(String::from("匿名"));

  

// 方式 3：map（對 Some 的值做轉換）

let upper = find_user(1).map(|name| name.to_uppercase());

  

// 方式 4：? 運算子（在函式中傳播 None）— 詳見 Result 章節

```

  

---

  

## 5. Result — 取代 Exception

  

```rust

// Result<T, E> 的定義：

// enum Result<T, E> {

//     Ok(T),

//     Err(E),

// }

  

use std::num::ParseIntError;

  

fn parse_age(s: &str) -> Result<u32, ParseIntError> {

    let age: u32 = s.trim().parse()?;  // ? 運算子：成功取值，失敗提早回傳 Err

    Ok(age)

}

  

// 處理 Result

match parse_age("25") {

    Ok(age) => println!("年齡: {}", age),

    Err(e) => println!("解析失敗: {}", e),

}

  

// ? 運算子：自動 early return Err

fn process(input: &str) -> Result<String, ParseIntError> {

    let n = input.trim().parse::<i32>()?;  // 失敗就提早回傳 Err

    Ok(format!("數值的兩倍是 {}", n * 2))

}

```

  

---

  

## 練習題

  

```bash

cargo run -p phase1_types --example ex01_shapes

cargo run -p phase1_types --example ex02_card_game

cargo run -p phase1_types --example ex03_error_handling

cargo test -p phase1_types

```

  

## 與 Java/Python 對比

  

| 概念 | Rust | Java | Python |

|------|------|------|--------|

| 資料結構 | `struct` + `impl` | `class` | `class` |

| 繼承 | 不支援（用 Trait 組合） | `extends` | 繼承 |

| null | 無，用 `Option<T>` | `null` | `None`（可能拋 AttributeError） |

| 例外 | 無，用 `Result<T,E>` | `throws Exception` | `raise Exception` |

| 枚舉值 | 可攜帶任意資料 | 只有常數 | `Enum` 類似常數 |