
## 🚀 給資深開發者的 Rust 快速通關指南

你已經熟悉了 Java、JavaScript/TypeScript 和 Python 等現代語言，這代表你對 OOP、非同步處理、型別系統都有深刻理解。

轉換到 Rust，你不需要從 `if/else` 或 `for` 迴圈學起，你只需要跨越幾個**底層思維的典範轉移 (Paradigm Shifts)**。以下是用你熟悉的語言視角，為你整理的核心觀念：

### 1. 記憶體管理：沒有 GC，也沒有手動 `free()`

這是 Rust 最震撼的設計。

- **在 Java / Python / V8 (JS)**：你一直依賴 Garbage Collector (GC) 在背景幫你掃描並回收記憶體。代價是不可預期的效能延遲 (GC Pause)。
    
- **在 C / C++**：你需要手動 `malloc` 和 `free`，稍有不慎就是 Memory Leak 或 Segmentation Fault。
    
- **在 Rust**：採用 **所有權 (Ownership)** 系統。
    
    當變數離開作用域 (Scope) 的那一瞬間（也就是遇到 `}` 時），編譯器會在**編譯階段**自動幫你安插釋放記憶體的程式碼。這也是為什麼 Tauri 可以做到極低記憶體消耗的原因。
    

### 2. 變數預設是不可變的 (Immutable by Default)

在 JS/TS 中，我們習慣用 `let` 宣告可變變數，`const` 宣告常數。

在 Rust 中，為了執行緒安全 (Thread-safety)，**所有變數預設都是唯讀的**。



```Rust
let x = 5; 
x = 6; // ❌ 編譯錯誤！x 預設不可變

let mut y = 5; // 必須顯式加上 mut (mutable)
y = 6; // ✅ 這樣才合法
```

### 3. 消滅 Null 參照：億萬美元的錯誤

在 Java 裡最怕遇到 `NullPointerException`；在 JS/TS 裡最煩的是 `Cannot read property of undefined`。

**Rust 的世界裡，完全沒有 `null` 這個關鍵字。**

取而代之的是，Rust 使用一個內建的泛型列舉 `Option<T>`。一個變數要麼有值 (`Some`)，要麼沒值 (`None`)。你**必須**用模式匹配 (Pattern Matching) 處理 `None` 的情況，否則連編譯都過不了。這從根本上消滅了 Null 錯誤。



```Rust
// 類似 TS 的: string | null
let maybe_name: Option<String> = Some(String::from("Kirin"));

match maybe_name {
    Some(name) => println!("名字是 {}", name),
    None => println!("沒有名字"), // 編譯器強迫你一定要寫這一行處理空值
}
```

### 4. 放棄 Try-Catch，擁抱 Result

Java 和 Python 高度依賴 `try { ... } catch (e) { ... }` 這種隱式的例外拋出機制。缺點是你光看函數簽名，根本不知道它到底會炸出什麼錯誤。

Rust 放棄了 Exceptions，採用 `Result<T, E>` 系統。函數執行成功回傳 `Ok(T)`，失敗回傳 `Err(E)`。



```Rust
// 這個函數宣告明白告訴你：成功回傳字串，失敗回傳 IO Error
fn read_file(path: &str) -> Result<String, std::io::Error> {
    // ...
}

// 呼叫端必須處理錯誤
let result = read_file("test.md");
match result {
    Ok(content) => println!("檔案內容: {}", content),
    Err(e) => println!("讀取失敗: {}", e),
}
```

### 5. 拋棄 Class 與繼承 (Inheritance)，改用組合 (Composition)

Rust 是一門強型別語言，但它**不是傳統的物件導向語言**。

這裡沒有 `class`，沒有 `extends`。資料與行為是徹底分離的：

- **`struct` (結構體)**：只負責定義資料欄位（類似 Java 的 POJO 或是 TS 的 `interface` 定義資料模型）。
    
- **`impl` (實作)**：專門為 `struct` 掛載方法。
    
- **`trait` (特徵)**：等同於 Java/TS 的 `interface`，定義共同行為。
    



```Rust
// 1. 定義資料
struct Note {
    title: String,
    content: String,
}

// 2. 定義行為
impl Note {
    fn new(title: &str, content: &str) -> Self {
        Note {
            title: title.to_string(),
            content: content.to_string(),
        }
    }
    
    fn print_summary(&self) {
        println!("標題: {}", self.title);
    }
}
```

---

只要大腦轉換過這五個觀念，你接下來看任何 Rust 程式碼都會有一種「原來如此」的痛快感，因為它的一切設計都是為了**在編譯期就把所有的潛在 Bug 抓出來**。

既然你已經成功運行了 Tauri 的前後端橋樑，我們下一個進度，要不要直接動手寫一支 Rust Command，實作 **讀取你電腦裡的 Markdown 檔案內容，並將字串回傳給 React 渲染**？這會剛好用到剛剛提到的 `Result` 和 `String` 處理！