# Java 熱替換：不重啟就換掉一段邏輯

在遊戲 server 或長跑服務裡，我們常想「改一段邏輯不重啟就生效」。Java 做得到，但要繞一圈。這篇整理兩條路線、實作模式，以及一定會踩的坑。

## 兩條路線

**第一條是 JDK 內建的 HotSwap（JVMTI / instrumentation）。** Debug 模式下可以 redefine 一個 class 的方法內部實作，IntelliJ、Eclipse 的 "Hot Swap" 就是走這條。但限制很硬：不能新增或刪除方法、欄位，不能改繼承結構或方法簽章。只能改「方法內的那幾行」。

**第二條是 ClassLoader 熱替換。** 如果你可以接受「舊 instance 不變、新 instance 用新版就好」，這條路更乾淨，也不用碰 instrumentation。以下主要講這條。

## ClassLoader 熱替換

核心模式是每次重載就開一個新的 `ClassLoader`：

```java
// 1. Runtime 編譯 .java → bytecode
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
compiler.run(null, null, null, "-d", "out/", "src/Plugin.java");

// 2. 每次重載開新 ClassLoader
URLClassLoader cl = new URLClassLoader(
    new URL[]{ new File("out/").toURI().toURL() },
    parentLoader  // 通常是 app classloader
);
Class<?> c = cl.loadClass("Plugin");
Object instance = c.getDeclaredConstructor().newInstance();
```

舊 class 之後被 GC，新物件走新版邏輯。既有 instance 不受影響。

## 一定會踩的坑

**必須有穩定的介面，而且它要放在 parent loader。** 同名 class 由不同 loader 載入 = 兩個不相容的型別，`instanceof` 和 cast 全掛。所以 interface 放 parent，實作類才放子 loader：

```java
// parent loader: interface Plugin { void run(); }
// child loader:  class PluginImpl implements Plugin
Plugin p = (Plugin) instance;  // 這樣才 work
```

**Parent-first delegation。** 介面要能從 parent 找到，別讓子 loader 自己載一份，否則又變成兩個不相容型別。

**舊 loader 要能被 GC。** 所有指向舊 instance 或舊 Class 的 reference 都要斷乾淨，否則 metaspace 洩漏。ThreadLocal、static cache、註冊過但沒反註冊的 listener 是最常見的洩漏源。

**static 狀態會重置。** 新 loader 有自己的一份 static 欄位——這通常正是你要的行為。

## 編譯到記憶體（進階）

想省 IO、也免掉 `out/` 目錄的並發問題，可以用 `JavaFileManager` + `SimpleJavaFileObject` 把 bytecode 收在記憶體的 `Map<String, byte[]>`，再讓自訂 ClassLoader 的 `findClass()` 直接 `defineClass()`。

## 效率

要分兩個時間點看。

**執行期：不輸原生。** 載完的東西就是正常 bytecode，一樣被 JIT 優化，跟平常沒差別。長跑的 hot loop 該熱的還是會熱。

**替換那一瞬間：貴。** 要跑 javac（幾十到幾百毫秒）、開 loader、重建 instance。所以「改一行、看效果」的高頻迴圈體感偏慢——這是最痛的地方，但只發生在替換當下，不是永久稅。

## 突破 HotSwap 限制的方案

如果需要加方法、加欄位、改繼承又不想自己刻 ClassLoader：

- **DCEVM / JetBrains Runtime**：patched JVM，支援結構性變更。
- **JRebel**：商業方案，最完整。
- **Spring Boot DevTools**：不是真 hotswap，是丟掉 app classloader 重建，快但會重置狀態。

## 實務建議

把「一次熱替換的單位」定成一個模組——一組 class 加一個 entry interface，每個模組一個 loader。這樣邊界清楚、GC 好回收。本質上就是 OSGi / JPMS layer 在做的事，只是自己刻更輕。