
以下是一篇**簡潔版教學 blog 草稿**，主題是介紹 `@JsonTypeInfo` 的三種模式：`Id.NAME`、`Id.CLASS`、`Id.MINIMAL_CLASS`。這版設計成一頁式技術速查教學，適合貼在團隊知識庫、Medium、Dev.to、HackMD 或 Notion。

---

# 🌟 精簡教學：用 `@JsonTypeInfo` 處理 Jackson 多型序列化的三種方式

在使用 Jackson 處理多型（polymorphic）資料時，我們常遇到這個問題：

```java
public class Zoo {
    public Animal animal;
}
```

`Animal` 是介面或抽象類別，JSON 無法知道具體類型是 `Dog` 還是 `Cat`，導致反序列化失敗。

別擔心，`@JsonTypeInfo` 幫你搞定！本文介紹三種最常用的類型標註方式：`Id.NAME`、`Id.CLASS`、`Id.MINIMAL_CLASS`。

---

## 🎯 1. `Id.NAME` – 安全且常用的白名單方式

### ✍️ 寫法：

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "dog"),
    @JsonSubTypes.Type(value = Cat.class, name = "cat")
})
public interface Animal {}
```

### 🧪 JSON 範例：

```json
{
  "type": "dog",
  "name": "Buddy",
  "bark": true
}
```

### ✅ 優點：

- 安全、簡潔
    
- 僅允許指定的子類別（白名單）
    

### ⚠️ 缺點：

- 每新增子類別需更新 `@JsonSubTypes`
    

---

## ⚡ 2. `Id.CLASS` – 自動儲存完整類別資訊

### ✍️ 寫法：

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.CLASS,
    include = JsonTypeInfo.As.PROPERTY,
    property = "@class"
)
public interface Animal {}
```

### 🧪 JSON 範例：

```json
{
  "@class": "com.example.Dog",
  "name": "Buddy",
  "bark": true
}
```

### ✅ 優點：

- 不用指定子類別，自動處理
    
- 適合通用封裝（如你要序列化任意型別）
    

### ⚠️ 缺點：

- 📛 **安全風險**：`Class.forName()` 可被注入任意類別
    
- 僅適用於可信環境或封閉系統
    

---

## ✨ 3. `Id.MINIMAL_CLASS` – 簡化版的 `CLASS` 路徑

### ✍️ 寫法：

```java
@JsonTypeInfo(
    use = JsonTypeInfo.Id.MINIMAL_CLASS,
    include = JsonTypeInfo.As.PROPERTY,
    property = "@class"
)
public interface Animal {}
```

### 🧪 JSON 範例：

```json
{
  "@class": ".Dog",
  "name": "Buddy",
  "bark": true
}
```

### ✅ 優點：

- JSON 更精簡
    
- 同樣自動反推類別
    

### ⚠️ 缺點：

- 更依賴 ObjectMapper 的基準 package
    
- 仍有類似 `Id.CLASS` 的安全問題
    

---

## 🧠 總結比較表

|模式|類別資訊|是否需註冊子類別|安全性|適合場景|
|---|---|---|---|---|
|`Id.NAME`|簡稱（"dog"）|✅ 需要|✅ 高|常見用法、安全環境|
|`Id.CLASS`|完整類別路徑|❌ 不需要|⚠️ 低|動態封裝、自動泛型轉型|
|`Id.MINIMAL_CLASS`|`.Dog`（簡寫）|❌ 不需要|⚠️ 低|精簡 JSON、內部使用為主|

---

## 🚀 結語

- 若你在做 API、前後端交換資料 👉 **推薦用 `Id.NAME` + 白名單註冊子類別**
    
- 若你在做內部工具、記錄動態型別資料 👉 **`Id.CLASS` / `MINIMAL_CLASS` 更方便**
    

Jackson 多型很強大，但也有安全與維護上的考量。選對 `@JsonTypeInfo` 的方式，才能寫出安全、可讀、好維護的序列化邏輯！

