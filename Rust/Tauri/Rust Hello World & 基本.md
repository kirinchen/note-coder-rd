建立一個全新的 Rust 專案並掌握核心語法，透過 Rust 內建的套件管理員 `Cargo` 可以非常快速地完成。以下是涵蓋建立專案、Hello World、控制流（if / for）、函式以及泛型（Generics）的實戰指南。

### 1. 創立專案 (Create Project)

打開終端機，使用 `cargo` 指令來建立並進入新的專案目錄：



```Bash
cargo new rust_tutorial
cd rust_tutorial
```

這會自動生成一個標準的目錄結構，其中 `Cargo.toml` 是專案的設定與依賴管理檔，而主要的程式碼會放在 `src/main.rs` 中。

### 2. Hello, World!

打開 `src/main.rs`，預設已經寫好了 Hello World：



```Rust
fn main() {
    // println! 結尾的驚嘆號代表這是一個巨集 (Macro) 而非一般函式
    println!("Hello, World!");
}
```

在終端機輸入以下指令即可編譯並執行：



```Bash
cargo run
```

### 3. 控制流：`if` 判斷式與 `for` 迴圈

Rust 的 `if` 條件不需要加括號，且 `if` 本身是一個表達式（Expression），可以將結果直接賦值給變數。迴圈方面，`for` 搭配疊代器（Iterator）或 Range 是最常見的寫法。



```Rust
fn main() {
    // --- if 判斷式 ---
    let system_status = 200;

    if system_status == 200 {
        println!("系統正常運作");
    } else if system_status == 404 {
        println!("找不到資源");
    } else {
        println!("未知狀態");
    }

    // if 可以作為表達式賦值
    let is_ok = true;
    let code = if is_ok { 1 } else { 0 };
    println!("代碼: {}", code);


    // --- for 迴圈 ---
    // 1. 透過 Range 執行迴圈 (1..5 包含 1 不包含 5；1..=5 包含 5)
    for i in 1..=3 {
        println!("Range 迴圈: {}", i);
    }

    // 2. 疊代陣列或集合
    let endpoints = ["/api/v1/users", "/api/v1/orders", "/api/v1/auth"];
    for endpoint in endpoints {
        println!("請求路由: {}", endpoint);
    }
}
```

### 4. 函式 (Functions)

Rust 使用 `fn` 宣告函式。參數必須明確標註型別（Type），如果函式有回傳值，必須使用 `->` 標示回傳型別。

**重點：** Rust 偏好省略 `return` 關鍵字，區塊中最後一行若「不加分號」，就會自動成為該區塊的回傳值。



```Rust
// 帶有參數與回傳值的函式
fn calculate_margin(revenue: i32, cost: i32) -> i32 {
    let profit = revenue - cost;
    
    // 不加分號結尾，等同於 return profit;
    profit 
}

fn main() {
    let result = calculate_margin(1000, 750);
    println!("利潤為: {}", result);
}
```

### 5. 泛型 (Generics)

泛型允許我們撰寫可重複使用的程式碼，而不必侷限於單一型別。在 Rust 中，泛型幾乎不會帶來執行期的效能損耗（因為編譯器會進行單態化 Monomorphization）。

#### 泛型函式

若要讓函式接收不同型別，我們在函式名稱後方加上 `<T>`。通常會搭配 `Trait`（特徵）來限制這個泛型必須具備什麼行為。



```Rust
// T: std::fmt::Debug 是一個 Trait Bound，
// 意思是「T 這個型別必須實作 Debug 特徵」，這樣才能被 {:?} 印出來。
fn print_debug_info<T: std::fmt::Debug>(data: T) {
    println!("Debug Info: {:?}", data);
}

fn main() {
    print_debug_info(404);               // 傳入整數
    print_debug_info("Connection lost"); // 傳入字串
    print_debug_info(vec![1, 2, 3]);     // 傳入 Vector
}
```

#### 泛型結構體 (Struct)

在定義資料結構時，泛型也非常實用。



```Rust
// 定義一個包含泛型的結構體
struct Response<T> {
    status: u16,
    data: T,
}

fn main() {
    // data 是字串
    let res_string = Response {
        status: 200,
        data: "Success",
    };

    // data 是整數
    let res_int = Response {
        status: 200,
        data: 1024,
    };

    println!("Status: {}, Data: {}", res_string.status, res_string.data);
}
```

熟悉這些基礎結構後，就可以進一步搭配 Cargo 引入第三方套件（Crates），開始建構更複雜的系統邏輯了。


## 中度進階

進入 Rust 的進階核心概念！`Result` 是 Rust 處理錯誤的靈魂，而巨集（Macro）則是 Rust 強大且靈活的元編程（Metaprogramming）武器。以下為你拆解這兩大重點：

### 1. 掌握 `Result` 型別

Rust 沒有傳統語言的 `try / catch` 異常處理機制。取而代之的是 `Result<T, E>` 這個列舉（Enum）。它強迫開發者在編譯階段就必須面對可能發生的錯誤，大幅提升程式的穩定性。

`Result` 有兩個變體（Variants）：

- `Ok(T)`：代表操作成功，並包含成功產生的值（型別為 `T`）。
    
- `Err(E)`：代表操作失敗，並包含錯誤資訊（型別為 `E`）。
    

#### 實戰範例：字串轉數字與 `?` 運算子



```Rust
use std::num::ParseIntError;

// 這個函式可能會失敗，所以回傳 Result
fn parse_and_double(s: &str) -> Result<i32, ParseIntError> {
    // 使用 ? 運算子：
    // 如果 parse 成功，會解開 Ok 裡的值放進變數 n。
    // 如果 parse 失敗，會提早 return Err 給呼叫者。
    let n = s.parse::<i32>()?; 
    
    // 成功的話，將結果包裝回 Ok
    Ok(n * 2)
}

fn main() {
    // 處理成功的情況
    match parse_and_double("42") {
        Ok(result) => println!("成功: {}", result),
        Err(e) => println!("錯誤: {}", e),
    }

    // 處理失敗的情況
    match parse_and_double("not_a_number") {
        Ok(result) => println!("成功: {}", result),
        Err(e) => println!("錯誤: {}", e),
    }
}
```

---

### 2. 巨集 (Macro) 與 函式 (Function) 的差別

雖然兩者都是用來封裝可重複使用的邏輯，但在底層運作上有著本質上的差異：

|**比較項目**|**函式 (Function)**|**巨集 (Macro)**|
|---|---|---|
|**執行時機**|**執行期 (Runtime)** 被呼叫。|**編譯期 (Compile time)** 展開為實際程式碼。|
|**參數數量**|必須固定，且必須明確標示型別。|**可變數量** (例如 `println!` 可以接任意數量的參數)。|
|**型別檢查**|嚴格，透過泛型處理不同型別。|展開後才進行檢查，可以操作語法樹（AST）。|
|**能力範圍**|執行運算、控制流。|可以**生成新的程式碼**、定義 DSL (領域特定語言)、自動實作 Trait 等。|
|**語法特徵**|名稱後不加驚嘆號 `my_func()`|名稱後必須加驚嘆號 `my_macro!()`|

**簡單來說：函式是「執行邏輯」，巨集是「寫程式碼的程式碼」。**

---

### 3. 如何寫巨集 (Declarative Macros)

Rust 中最常用、也最容易上手的巨集是 **宣告式巨集 (Declarative Macros)**，透過 `macro_rules!` 來定義。它的核心概念是「模式匹配（Pattern Matching）」。

我們來寫一個實用的巨集：`debug_var!`，它會自動印出變數的名稱和數值（類似 JavaScript 的 `console.log({var_name})` 效果）。

#### 實戰範例：自訂巨集



```Rust
// 定義巨集
macro_rules! debug_var {
    // 這裡定義匹配的模式
    // $val 是一個變數名稱，expr 代表它匹配的是一個「表達式 (Expression)」
    ($val:expr) => {
        // 展開後的程式碼：stringify! 是內建巨集，會把傳入的變數名稱轉成字串
        println!("變數 [{}] 的值為: {:?}", stringify!($val), $val);
    };
}

// 支援多個參數的進階版本（使用重複模式）
macro_rules! make_vec {
    // $( ... ),* 意思是匹配多個由逗號分隔的表達式
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            // $( ... )* 代表針對每個匹配到的值，重複產生這段程式碼
            $(
                temp_vec.push($x);
            )*
            temp_vec // 回傳向量
        }
    };
}

fn main() {
    // 測試我們寫的 debug_var!
    let server_port = 8080;
    let api_key = "sk_test_123";
    
    debug_var!(server_port); // 展開成: println!("變數 [{}] ...", "server_port", server_port);
    debug_var!(api_key);

    // 測試我們寫的 make_vec!
    let my_list = make_vec![1, 2, 3, 4, 5];
    println!("自訂巨集產生的 Vector: {:?}", my_list);
}
```

**巨集匹配符號（Designators）小錦囊：**

撰寫 `macro_rules!` 時，常用的匹配型別有：

- `expr`：表達式 (例如 `2 + 2`, `"hello"`, `x`)
    
- `ident`：識別字 (例如變數名稱、函式名稱)
    
- `ty`：型別 (例如 `i32`, `String`)
    
- `stmt`：陳述式 (Statement)
    

_(註：Rust 還有另一種更進階的巨集叫做「程序集巨集 Procedural Macros」，常用於 `#[derive(Debug)]` 等屬性標籤。但它需要建立獨立的 Crate 並操作 Token Stream，通常在開發大型框架如 Serde、Rocket 時才會自己撰寫。)_