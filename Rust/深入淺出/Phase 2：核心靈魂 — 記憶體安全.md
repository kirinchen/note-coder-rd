# Phase 2：核心靈魂 — 記憶體安全

  

> 這是 Rust 最獨特、最重要的部分。

> 理解所有權系統後，你才真正「懂 Rust」。

  

---

  

## 為什麼需要所有權系統？

  

| 語言 | 記憶體管理方式 | 問題 |

|------|--------------|------|

| C/C++ | 手動管理（malloc/free） | Use-after-free、double-free、記憶體洩漏 |

| Java/Python | 垃圾回收（GC） | GC pause、執行時開銷、無法預測回收時機 |

| **Rust** | **所有權系統（編譯期保證）** | **零執行時開銷，且保證無記憶體問題** |

  

Rust 的目標：**不靠 GC，在編譯期就消除所有記憶體錯誤。**

  

---

  

## 1. Ownership 所有權

  

### 三大規則（必須背起來）

  

1. Rust 中每個值都有一個「擁有者」（owner）

2. 每個值同一時間只能有**一個**擁有者

3. 當擁有者離開 scope，該值就被**自動釋放**（呼叫 `drop`）

  

```rust

{

    let s = String::from("hello");  // s 是這個 String 的 owner

    // 可以使用 s

}  // ← 這裡 s 的 scope 結束，String 被自動釋放（drop）

   // 不需要 free()，也沒有 GC

```

  

### Move 語意（所有權轉移）

  

```rust

let s1 = String::from("hello");

let s2 = s1;    // s1 的所有權「移動」給 s2

  

// println!("{}", s1);  // ❌ 編譯錯誤！s1 已失效

println!("{}", s2);     // ✅ OK，s2 現在是 owner

```

  

> **與 Java 的差異：** Java 的 `String s2 = s1` 是複製「參考」，s1 和 s2 都指向同一個物件。

> Rust 的 String 是「移動」，s1 失效，只有 s2 有效。

> 這樣就能保證：同一塊記憶體不會被 drop 兩次。

  

### Copy 型別（例外）

  

```rust

// 基本型別（整數、浮點、bool、char）存在 Stack 上，複製成本極低

// 它們實作了 Copy trait，賦值時是複製而不是移動

let x = 5;

let y = x;  // x 被複製，x 和 y 都有效

println!("{} {}", x, y);  // ✅ 都 OK

  

// Heap 上的資料（String、Vec 等）預設是 Move，不是 Copy

```

  

---

  

## 2. Borrowing 借用

  

如果所有值都只能有一個 owner，程式會很難寫。

**Borrowing（借用）** 讓你在不取得所有權的情況下「使用」一個值。

  

### 不可變借用（&T）

  

```rust

fn calculate_length(s: &String) -> usize {  // 借用 s，不取得所有權

    s.len()

}  // s 超出 scope，但不 drop（因為我們只是借用）

  

let s = String::from("hello");

let len = calculate_length(&s);  // &s 是對 s 的借用

println!("'{}' 長度是 {}", s, len);  // s 仍然有效

```

  

### 可變借用（&mut T）

  

```rust

fn add_word(s: &mut String) {

    s.push_str(", world");

}

  

let mut s = String::from("hello");

add_word(&mut s);

println!("{}", s);  // "hello, world"

```

  

### 借用規則（編譯器強制執行）

  

**在同一個 scope 中：**

- 可以有**任意多個**不可變借用（`&T`）

- 或者**只能有一個**可變借用（`&mut T`）

- **不能同時存在**不可變和可變借用

  

```rust

let mut s = String::from("hello");

  

let r1 = &s;     // ✅ 不可變借用 1

let r2 = &s;     // ✅ 不可變借用 2（可以多個）

// let r3 = &mut s;  // ❌ 不能同時有可變借用

  

println!("{} {}", r1, r2);

// r1, r2 的生命週期在這裡結束

  

let r3 = &mut s;  // ✅ 現在可以了，r1/r2 已不再使用

r3.push_str("!");

```

  

> **為什麼這麼設計？** 防止 Data Race（資料競爭）。

> 多個讀取是安全的，但同時讀取 + 寫入就有問題。

  

---

  

## 3. String vs &str — 新手最常卡關的地方

  

```

 Stack                    Heap

┌─────────────────┐      ┌───────────────────┐

│  String         │      │  h e l l o w o r  │

│  ┌───────────┐  │      │  l d              │

│  │ ptr ──────┼──┼─────→│                   │

│  │ len: 10   │  │      └───────────────────┘

│  │ cap: 10   │  │

│  └───────────┘  │

│                 │

│  &str           │

│  ┌───────────┐  │

│  │ ptr ──────┼──┼─────→（指向 Heap 或字串常量區）

│  │ len: 5    │  │

│  └───────────┘  │

└─────────────────┘

```

  

| | `String` | `&str` |

|--|---------|--------|

| 所有權 | 擁有資料 | 借用資料 |

| 可變性 | 可增刪（push、pop） | 不可變 |

| 儲存 | Heap | 指向任意位置 |

| 轉換 | `"hello".to_string()` | `s.as_str()` 或 `&s` |

  

```rust

// 字串字面量 = &str，儲存在程式的二進位檔中

let s1: &str = "hello";  // 'static lifetime

  

// String = Heap 上可變的字串

let mut s2: String = String::from("hello");

s2.push_str(", world");

  

// &String 可以自動轉成 &str（Deref coercion）

fn print_str(s: &str) {

    println!("{}", s);

}

  

print_str("直接字面量");  // &str

print_str(&s2);           // &String 自動轉 &str ← 很常用的技巧

```

  

---

  

## 4. Lifetimes 生命週期（進階）

  

```rust

// 這個函式的回傳值借自哪裡？

// 'a 標記說明：回傳的 &str 和輸入的 &str 有相同的生命週期

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {

    if x.len() > y.len() { x } else { y }

}

```

  

大多數情況下，Rust 編譯器可以自動推斷 lifetime（**elision rules**），不需要手動標記。

只有在編譯器無法推斷時才需要明確標記。

  

---

  

## 練習題

  

```bash

cargo run -p phase2_memory --example ex01_ownership_moves

cargo run -p phase2_memory --example ex02_borrowing_rules

cargo run -p phase2_memory --example ex03_string_operations

cargo test -p phase2_memory

```