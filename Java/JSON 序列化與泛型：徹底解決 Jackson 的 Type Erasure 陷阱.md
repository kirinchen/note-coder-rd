
# [Java 實戰] JSON 序列化與泛型：徹底解決 Jackson 的 Type Erasure 陷阱

在 Java 開發中，使用 Jackson 處理 JSON 是家常便飯。但你一定遇過這個經典場景：你快樂地把一個 `List<User>` 序列化成 JSON，存到 Redis 或資料庫，等你把它反序列化（Deserialize）回來時，以為會拿到 `List<User>`，結果一跑 `get(0)` 卻噴出了 `ClassCastException`。

仔細一看，發現原本的 `User` 物件變成了 `LinkedHashMap`。

這就是 Java **泛型擦除 (Type Erasure)** 帶來的惡夢。本篇攻略將帶你由淺入深，解決從「編譯期」到「執行期」，甚至「持久化」的泛型 JSON 處理難題。

---

## 為什麼會變成 LinkedHashMap？

Java 的泛型只存在於編譯時期。當你執行 `List<User>` 時，JVM 在 Runtime 只看得到 `List`（Raw Type）。

如果你寫：



```Java
// 錯誤示範
List<User> users = mapper.readValue(json, List.class);
```

Jackson 看到 `List.class`，它不知道裡面裝的是什麼，所以它會用最安全的預設值：`java.util.LinkedHashMap` 來存放 JSON 物件。這就是災難的開始。

---

## 攻略一：編譯期已知型別 (The Standard Way)

這是最基本的情況。如果你在寫程式碼的當下，已經明確知道你要轉回來的是 `List<User>`，請使用 **`TypeReference`**。

它利用了 Java 的匿名內部類別（Anonymous Inner Class）機制來「騙」過編譯器保留泛型資訊。



```Java
import com.fasterxml.jackson.core.type.TypeReference;

String json = "[{\"name\":\"Bzk\", \"age\":18}]";

// 正確做法：使用 TypeReference 捕捉泛型
List<User> users = mapper.readValue(json, new TypeReference<List<User>>(){});

// 現在你可以安心地操作 User 物件了
System.out.println(users.get(0).getName());
```

---

## 攻略二：執行期動態型別 (The Dynamic Way)

這是很多框架開發者的痛點。假設你寫了一個通用方法，參數傳進來的是一個 `Class<T>`，你要回傳 `List<T>`。這時候你不能寫 `new TypeReference<List<T>>(){}`，因為 `T` 是一個變數。

這時，Jackson 的 **`TypeFactory`** 就是你的救星。它可以像樂高一樣，動態組裝出你需要的型別。

### 場景：我有 List 裡的 Class 物件



```Java
public <T> List<T> parseList(String json, Class<T> elementClass) {
    ObjectMapper mapper = new ObjectMapper();
    
    // 透過 TypeFactory 動態建構 "List<elementClass>"
    // constructCollectionType(容器型別, 內容物型別)
    JavaType listType = mapper.getTypeFactory()
        .constructCollectionType(List.class, elementClass);
        
    // readValue 支援直接傳入 JavaType
    return mapper.readValue(json, listType);
}

// 使用方式
List<User> users = parseList(jsonString, User.class);
```

### 場景：我有更複雜的包裝類別 (e.g., Result<T>)

如果你的結構是 `Result<Data<T>>` 這種多層泛型：



```Java
JavaType innerType = mapper.getTypeFactory()
    .constructParametricType(Data.class, elementClass); // Data<T>

JavaType finalType = mapper.getTypeFactory()
    .constructParametricType(Result.class, innerType);  // Result<Data<T>>
```

---

## 攻略三：型別資訊的持久化 (The Storage Way)

這是最高級的應用。假設你的需求是：**「我要把物件存進 DB，同時我也要把它的泛型資訊存起來，下次讀出來時，程式完全不知道它是什麼，但要能完美還原。」**

我們不希望在 JSON 裡面塞滿醜陋的 `@class` (Jackson Default Typing)，我們希望 JSON 保持乾淨，而將型別資訊分開儲存。

Jackson 支援將 `JavaType` 轉換為 **Canonical String (規範化字串)**。

### 1. 儲存 (Serialize)



```Java
// 假設我們有一個複雜型別 List<User>
JavaType type = mapper.getTypeFactory()
    .constructCollectionType(List.class, User.class);

// 1. 將資料轉為 JSON
String jsonData = mapper.writeValueAsString(userList);

// 2. 將型別轉為字串 (這個字串可以存到 DB 的另一個欄位)
String typeInfo = type.toCanonical();

System.out.println("儲存的型別: " + typeInfo);
// 輸出範例: "java.util.List<net.bzk.User>"
```

### 2. 還原 (Deserialize)

當你從 DB 讀出這兩串字串時：



```Java
String jsonData = ...; // 從 DB 讀取的 JSON
String typeInfo = ...; // 從 DB 讀取的 "java.util.List<net.bzk.User>"

// 1. 透過字串還原 JavaType
JavaType restoredType = mapper.getTypeFactory().constructFromCanonical(typeInfo);

// 2. 反序列化
List<User> list = mapper.readValue(jsonData, restoredType);
```

這樣做的好處是：**你的 JSON 資料完全純淨（不綁定 Java 實作細節），但你依然擁有強型別的反序列化能力。**

---

## 總結 Cheat Sheet

|**場景**|**關鍵技術**|**程式碼片段**|
|---|---|---|
|**寫死型別**|`TypeReference`|`new TypeReference<List<User>>(){}`|
|**動態傳入 Class**|`TypeFactory`|`factory.constructCollectionType(List.class, clazz)`|
|**複雜包裝類**|`TypeFactory`|`factory.constructParametricType(Wrapper.class, innerType)`|
|**存儲/還原泛型**|`Canonical String`|`type.toCanonical()` / `factory.constructFromCanonical(str)`|

掌握 `TypeFactory`，你就掌握了 Jackson 處理泛型的核心。下次再看到 `LinkedHashMap` 出現在你的 List 裡，你就知道該怎麼辦了！

---

_Happy Coding!_