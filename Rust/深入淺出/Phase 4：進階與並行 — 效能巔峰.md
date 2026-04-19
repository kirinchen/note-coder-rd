# Phase 4：進階與並行 — 效能巔峰

  

> 目標：掌握 Rust 的進階記憶體管理、執行緒安全、以及非同步程式設計。

  

---

  

## 1. Smart Pointers（智慧指標）

  

### 為什麼需要 Smart Pointers？

  

Rust 的所有權規則很嚴格：一個值只能有一個擁有者。

但有時你需要：

- 在 Heap 上分配資料（`Box<T>`）

- 多個 owner 共享同一個值（`Rc<T>` / `Arc<T>`）

- 內部可變性（在不可變引用中修改資料）（`RefCell<T>`）

  

### Box\<T\> — Heap 分配

  

```rust

// 最簡單的 Smart Pointer：強制把資料放到 Heap 上

let b = Box::new(5);        // 5 存在 Heap，b 是 Stack 上的指標

println!("{}", b);           // 透明地使用，就像普通值

  

// 用途 1：遞迴資料結構（大小在編譯期未知）

enum List {

    Cons(i32, Box<List>),   // 沒有 Box 的話，編譯器無法知道大小

    Nil,

}

  

// 用途 2：Trait Object（dyn Trait）

let animal: Box<dyn Animal> = Box::new(Dog::new("旺財"));

```

  

### Rc\<T\> — 多重所有權（單執行緒）

  

```rust

use std::rc::Rc;

  

let a = Rc::new(String::from("shared"));

let b = Rc::clone(&a);    // 增加引用計數，不是深拷貝

let c = Rc::clone(&a);    // 再增加一個

  

println!("引用計數: {}", Rc::strong_count(&a));  // 3

  

// 當所有 Rc 都 drop 後，資料才釋放

// ⚠️ Rc 不能跨執行緒（不是 Send），執行緒安全用 Arc

```

  

### Arc\<T\> — 原子計數（多執行緒）

  

```rust

use std::sync::Arc;

use std::thread;

  

let data = Arc::new(vec![1, 2, 3]);

let data2 = Arc::clone(&data);

  

let handle = thread::spawn(move || {

    println!("{:?}", data2);  // 另一個執行緒使用

});

```

  

### RefCell\<T\> — 內部可變性

  

```rust

use std::cell::RefCell;

  

// 繞過編譯期借用檢查，改為執行期檢查

let x = RefCell::new(5);

  

{

    let mut val = x.borrow_mut();  // 執行期可變借用

    *val += 1;

}  // borrow_mut 在這裡 drop

  

println!("{}", x.borrow());  // 6

```

  

### Arc + Mutex — 多執行緒共享可變狀態

  

```rust

use std::sync::{Arc, Mutex};

  

let counter = Arc::new(Mutex::new(0));

let mut handles = vec![];

  

for _ in 0..10 {

    let c = Arc::clone(&counter);

    let h = thread::spawn(move || {

        let mut num = c.lock().unwrap();

        *num += 1;

    });

    handles.push(h);

}

for h in handles { h.join().unwrap(); }

println!("最終計數: {}", *counter.lock().unwrap());  // 10

```

  

---

  

## 2. Concurrency（並行）

  

### Send 和 Sync Trait

  

```rust

// Send：型別可以安全地「移動」到另一個執行緒

// Sync：型別可以安全地「共享引用」到另一個執行緒

  

// Rc<T>: 不是 Send（執行緒不安全的引用計數）

// Arc<T>: 是 Send（原子計數）

// RefCell<T>: 是 Send 但不是 Sync

// Mutex<T>: 是 Send 也是 Sync

```

  

### Channel — 執行緒間通訊

  

```rust

use std::sync::mpsc;  // multiple producer, single consumer

  

let (tx, rx) = mpsc::channel();

  

thread::spawn(move || {

    tx.send("hello from thread").unwrap();

});

  

let msg = rx.recv().unwrap();

println!("{}", msg);

```

  

---

  

## 3. Async/Await（非同步）

  

### 基本概念

  

```rust

// async fn 回傳 Future（懶執行，需要 executor 驅動）

async fn fetch_data(url: &str) -> Result<String, reqwest::Error> {

    let body = reqwest::get(url).await?.text().await?;

    Ok(body)

}

  

// Tokio 是最流行的 async runtime

#[tokio::main]

async fn main() {

    let data = fetch_data("https://example.com").await;

    println!("{:?}", data);

}

```

  

### async vs thread

  

| | `thread` | `async/await` |

|--|---------|--------------|

| 適合 | CPU 密集 | I/O 密集 |

| 開銷 | 較大（OS thread） | 較小（綠色執行緒） |

| 語法 | spawn + join | async/await |

| 使用場景 | 並行計算 | 網路服務、資料庫 |

  

---

  

## 練習題

  

```bash

# Smart Pointers

cargo run -p phase4_advanced --example ex01_smart_pointers

  

# 執行緒並行

cargo run -p phase4_advanced --example ex02_concurrency

  

# Async/Await（需要 tokio）

cargo run -p phase4_advanced --example ex03_async

  

cargo test -p phase4_advanced

```