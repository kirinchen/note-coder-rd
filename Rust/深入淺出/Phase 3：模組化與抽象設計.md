# Phase 3：模組化與抽象設計

  

> 目標：學習 Rust 的抽象機制，寫出高重用性且結構清晰的程式碼。

  

---

  

## 1. Traits（特徵）— 共享行為的契約

  

Trait 類似 Java 的 Interface，但功能更強大：

- 可以有預設實作

- 可以當作泛型的約束條件（trait bounds）

- 可以動態派送（`dyn Trait`）

  

### 定義與實作 Trait

  

```rust

trait Describable {

    // 必須實作的方法（無預設實作）

    fn describe(&self) -> String;

  

    // 有預設實作的方法（可以覆寫）

    fn short_description(&self) -> String {

        let full = self.describe();

        if full.len() > 50 {

            format!("{}...", &full[..50])

        } else {

            full

        }

    }

}

  

struct Dog { name: String, breed: String }

struct Cat { name: String, indoor: bool }

  

impl Describable for Dog {

    fn describe(&self) -> String {

        format!("{}是一隻{}犬", self.name, self.breed)

    }

}

  

impl Describable for Cat {

    fn describe(&self) -> String {

        let kind = if self.indoor { "室內" } else { "室外" };

        format!("{}是一隻{}貓", self.name, kind)

    }

}

```

  

### Trait Bounds（泛型約束）

  

```rust

// 函式接受「任何實作了 Describable 的型別」

fn print_description<T: Describable>(item: &T) {

    println!("{}", item.describe());

}

  

// 多個 Trait bounds

fn print_debug_and_describe<T: Describable + std::fmt::Debug>(item: &T) {

    println!("{:?} -> {}", item, item.describe());

}

  

// where 語法（當 bounds 很長時更清晰）

fn complex<T>(item: &T)

where

    T: Describable + Clone + std::fmt::Debug,

{

    // ...

}

```

  

### 動態派送（dyn Trait）

  

```rust

// 編譯期決定型別（靜態派送，零額外開銷）

fn static_dispatch<T: Describable>(item: &T) { ... }

  

// 執行期決定型別（動態派送，有 vtable 開銷，但更靈活）

fn dynamic_dispatch(item: &dyn Describable) { ... }

  

// 可以把不同型別的物件放進同一個 Vec！

let animals: Vec<Box<dyn Describable>> = vec![

    Box::new(Dog { name: "旺財".to_string(), breed: "柴犬".to_string() }),

    Box::new(Cat { name: "咪咪".to_string(), indoor: true }),

];

```

  

---

  

## 2. Generics（泛型）

  

```rust

// 泛型函式

fn largest<T: PartialOrd>(list: &[T]) -> &T {

    let mut largest = &list[0];

    for item in list {

        if item > largest {

            largest = item;

        }

    }

    largest

}

  

// 泛型結構體

struct Pair<T> {

    first: T,

    second: T,

}

  

impl<T: std::fmt::Display + PartialOrd> Pair<T> {

    fn cmp_display(&self) {

        if self.first >= self.second {

            println!("較大的是: {}", self.first);

        } else {

            println!("較大的是: {}", self.second);

        }

    }

}

```

  

---

  

## 3. Module System（模組系統）

  

```rust

// src/main.rs 或 src/lib.rs

mod math {

    pub fn add(x: i32, y: i32) -> i32 { x + y }

  

    pub mod advanced {

        pub fn power(base: i32, exp: u32) -> i32 {

            base.pow(exp)

        }

    }

}

  

use math::advanced::power;   // 把 power 帶入 scope

  

fn main() {

    println!("{}", math::add(1, 2));

    println!("{}", power(2, 10));

}

```

  

### 多檔案模組

  

```

src/

├── main.rs          ← mod math; 宣告

├── math.rs          ← math 模組的實作（或 math/mod.rs）

└── math/

    ├── mod.rs       ← math 模組的 re-export

    └── advanced.rs  ← math::advanced 子模組

```

  

### Cargo 依賴管理

  

```toml

# Cargo.toml

[dependencies]

serde = { version = "1.0", features = ["derive"] }

tokio = { version = "1", features = ["full"] }

```

  

---

  

## 練習題

  

```bash

cargo run -p phase3_modules --example ex01_traits

cargo run -p phase3_modules --example ex02_generics

cargo run -p phase3_modules --example ex03_iterator_chain

cargo test -p phase3_modules

```