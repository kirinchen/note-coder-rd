---
title: Java ScopedValue 取代 ThreadLocal 的新選擇
tags: [java, thread, virtual-thread]

---

# Java ScopedValue 取代 ThreadLocal 的新選擇

## TL;DR

`ScopedValue` 是 JDK 在 Project Loom 下推出的新 API，用來在「同一條呼叫鏈（call chain）」裡共享 **不可變（immutable）** 資料，目的是取代許多 `ThreadLocal` 的使用情境。

- **不可變**：綁定之後在該 scope 內不能被改寫，要換值只能往下開新的 scope。
- **生命週期明確**：值只在 `run()` / `call()` 的 lambda 執行期間有效，跳出 scope 自動消失，不需要也不能 `remove()`。
- **對 Virtual Thread 友善**：百萬條虛擬執行緒下記憶體成本極低，繼承機制走 `StructuredTaskScope` 而不是複製。

> JEP 演進：JEP 429 → 446 → 464 → 481 → 487。在 JDK 21~24 都還是 Preview，使用時需要加上 `--enable-preview`。請以你的 JDK 版本實際狀態為準。

---

## 為什麼需要它？ThreadLocal 的三個痛點

`ThreadLocal` 長年被用來做「隱性傳參」，例如使用者身分、交易 ID、語系等不想一路 `method(... , ctx)` 傳下去的東西。但它有幾個老問題：

1. **可變且無邊界**：任何拿得到 `ThreadLocal` 的程式都能 `set()`，值能存活到整個執行緒生命週期，導致資料什麼時候被改、什麼時候該清不明確。
2. **記憶體洩漏風險**：執行緒池（ThreadPool）會重用執行緒，忘記 `remove()` 就會殘留舊值，甚至造成 leak。
3. **繼承成本高**：`InheritableThreadLocal` 在每次建立子執行緒時都會「複製」一份 map。對動輒上百萬的 Virtual Thread 來說，這個成本是災難。

`ScopedValue` 用「不可變 + 明確 scope」這兩個設計，把上面的問題一次解掉。

---

## 基本用法

### 宣告與綁定

```java=
import java.lang.ScopedValue;

public class Server {
    // 慣例：宣告成 static final
    private static final ScopedValue<String> CURRENT_USER = ScopedValue.newInstance();

    void handleRequest(String user) {
        // 在這個 scope 內 CURRENT_USER == user
        ScopedValue.where(CURRENT_USER, user)
                   .run(() -> processRequest());
        // 跳出 run() 之後，CURRENT_USER 自動失效
    }

    void processRequest() {
        // 不用一路把 user 當參數傳進來，這裡直接拿
        log();
    }

    void log() {
        // 呼叫鏈再深也讀得到
        System.out.println("current user = " + CURRENT_USER.get());
    }
}
```

重點：
- `where(key, value)` 回傳一個 `Carrier`，只有在 `run()` / `call()` 的 lambda 內，`get()` 才拿得到值。
- 沒綁定就 `get()` 會丟 `NoSuchElementException`，可先用 `isBound()` 判斷。

### 有回傳值用 call()

```java=
String result = ScopedValue.where(CURRENT_USER, "kirin")
                           .call(() -> doSomething());
```

`call()` 允許 lambda 丟出 checked exception，`run()` 則用於沒有回傳值的情況。

### 一次綁定多個值

```java=
ScopedValue.where(CURRENT_USER, "kirin")
           .where(TRACE_ID, "abc-123")
           .run(() -> handle());
```

---

## ThreadLocal vs ScopedValue 對照

| 面向 | ThreadLocal | ScopedValue |
|------|-------------|-------------|
| 可變性 | 可隨時 `set()` 改值 | 不可變，綁定後唯讀 |
| 生命週期 | 整條執行緒，需手動 `remove()` | 只在 scope（lambda）內，自動回收 |
| 重綁值 | 直接覆蓋 | 不能覆蓋，只能往下開新 scope（rebinding）|
| 繼承到子執行緒 | `InheritableThreadLocal`，複製整份 map | 配合 `StructuredTaskScope` 自動、零複製繼承 |
| Virtual Thread | 記憶體成本高 | 成本極低 |
| 洩漏風險 | 高（忘記 remove） | 幾乎沒有 |

---

## Rebinding：在子 scope 暫時換值

因為不可變，要「臨時換值」的方式不是覆寫，而是再開一層新的 scope：

```java=
private static final ScopedValue<String> NAME = ScopedValue.newInstance();

void outer() {
    ScopedValue.where(NAME, "outer").run(() -> {
        System.out.println(NAME.get());   // outer

        // 在內層暫時改成 inner
        ScopedValue.where(NAME, "inner").run(() -> {
            System.out.println(NAME.get()); // inner
        });

        System.out.println(NAME.get());   // outer（內層結束後恢復）
    });
}
```

這種「結構化」的覆寫讓值的變化範圍一目了然，不會像 `ThreadLocal.set()` 那樣到處污染。

---

## 搭配 Virtual Thread / StructuredTaskScope

`ScopedValue` 真正的威力在於和結構化並行（Structured Concurrency）一起用。父執行緒綁定的值，會自動被 `fork()` 出來的子任務「繼承」，而且不需要複製：

```java=
private static final ScopedValue<String> USER = ScopedValue.newInstance();

void handle() throws Exception {
    ScopedValue.where(USER, "kirin").run(() -> {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            // 兩個子任務都讀得到 USER == "kirin"
            scope.fork(() -> queryOrders());
            scope.fork(() -> queryProfile());

            scope.join().throwIfFailed();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    });
}

String queryOrders() {
    return "orders of " + USER.get();   // kirin
}
```

對比 `InheritableThreadLocal` 每開一條執行緒就複製 map，`ScopedValue` 走的是「子任務直接引用父 scope」，在百萬虛擬執行緒下幾乎沒有額外開銷。

---

## 常見實務範例：把 Request 身分隱性帶下去

```java=
public class RequestContext {
    public static final ScopedValue<User> USER = ScopedValue.newInstance();
    public static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();

    private RequestContext() {}
}

// Filter / Interceptor 入口
void doFilter(HttpRequest req, Runnable chain) {
    User user = authenticate(req);
    String traceId = req.header("X-Trace-Id");

    ScopedValue.where(RequestContext.USER, user)
               .where(RequestContext.TRACE_ID, traceId)
               .run(chain);
}

// Service 層任何深處
void businessLogic() {
    User u = RequestContext.USER.get();
    audit(RequestContext.TRACE_ID.get(), u);
}
```

跟以前用 `ThreadLocal` 寫 context holder 比起來，少了 `try { set } finally { remove }` 的樣板，也不會忘記清理。

---

## 注意事項 / 踩雷

- **目前仍是 Preview**：編譯與執行都要加 `--enable-preview`，且 `--release` 需指定對應版本。
  ```bash
  javac --release 24 --enable-preview Server.java
  java --enable-preview Server
  ```
- **不要拿 ScopedValue 存可變狀態**：它的設計前提就是不可變。要「在 scope 內累積結果」請用其他機制（例如回傳值、`StructuredTaskScope` 的子任務結果）。
- **`get()` 前注意是否已綁定**：未綁定會丟 `NoSuchElementException`，必要時先 `if (KEY.isBound())`。
- **無法跨 scope 共享改動**：內層改了值，外層看不到（這正是它的優點）。需要往上回傳請走正規 return。

---

## 小結

| 情境 | 建議 |
|------|------|
| 需要在呼叫鏈隱性共享「唯讀」資料 | ✅ `ScopedValue` |
| 大量使用 Virtual Thread | ✅ `ScopedValue` |
| 真的需要在執行緒內可變、長生命週期狀態 | 還是 `ThreadLocal` |

一句話總結：**只要你的 `ThreadLocal` 用法是「設一次、唯讀、用完即丟」，就可以改用 `ScopedValue`**，更安全、更省記憶體，也更適合 Loom 的世界。

---

###### tags: `java` `thread` `virtual-thread`
