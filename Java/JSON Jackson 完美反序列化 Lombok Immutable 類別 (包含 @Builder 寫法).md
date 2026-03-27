
## 讓 Jackson 完美反序列化 Lombok Immutable 類別 (包含 @Builder 寫法)

在 Java 開發中，將資料傳輸物件 (DTO) 設計為**不可變 (Immutable)** 是一個極佳的實踐。通常我們會使用 `final` 欄位來防止資料被意外修改。

但當你嘗試用 Jackson 將 JSON 字串反序列化回這個物件時，通常會立刻收到一個 `InvalidDefinitionException` 錯誤。

**為什麼會報錯？**

Jackson 預設的反序列化機制是：先找**無參數建構子 (No-args Constructor)** 建立空物件，再透過 Setter 塞值。

因為我們的類別全是 `final` 欄位，沒有無參數建構子也沒有 Setter，Jackson 不知道該如何把 JSON 的 Key 對應到物件上。

過去我們必須寫滿 `@JsonCreator` 和 `@JsonProperty` 來解決，這會讓程式碼變得很醜。現在，我們有兩種更現代、更優雅的解法！

---

### 解法一：極簡建構子派 (使用 `ParameterNamesModule`)

如果你喜歡最純粹的 Java 類別，不想要 Builder 模式，這招最適合你。

**1. 極簡的資料類別**

只需要 Lombok 的 `@RequiredArgsConstructor`，不需要任何 Jackson 註解：



```Java
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
public class UserProfile {
    public final String account;
    public final String email;
    public final int age;
}
```

**2. 核心設定：註冊模組**

在你的 `ObjectMapper` 中註冊 `ParameterNamesModule`，並確保編譯時有開啟 `-parameters` 參數（Spring Boot 預設已開啟）：

Java

```
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.module.paramnames.ParameterNamesModule;

public class JsonUtils {
    // 註冊 ParameterNamesModule 是魔法生效的關鍵
    public static final ObjectMapper JSON_MAPPER = new ObjectMapper()
            .registerModule(new ParameterNamesModule()); 
}
```

---

### 解法二：流暢建造者派 (使用 `@Builder` + `@Jacksonized`)

當你的 DTO 欄位很多，或者有很多非必填的選填欄位時，我們通常會加上 `@Builder`。但 Jackson 預設並不認識 Lombok 產生出來的 Builder。這時，`@Jacksonized` 就是你的救星！

**資料類別寫法：**

同時加上 `@Builder` 和 `@Jacksonized`，Jackson 就會自動切換為「透過 Builder 來反序列化」的模式。



```Java
import lombok.Builder;
import lombok.extern.jackson.Jacksonized;

@Jacksonized
@Builder
public class VectorProfile {
    public final double fromYYR;
    public final double deltaYYR;
    public final double deltaDurationRatio;
    
    // 如果有非必填欄位，Builder 模式處理起來比建構子優雅得多
    public final String optionalDescription; 
}
```

**這個寫法的優點：**

1. **無須修改 ObjectMapper**：這個寫法不需要像解法一那樣去註冊 `ParameterNamesModule` 或依賴編譯器參數，它是自帶 Jackson 註解的標準做法。
    
2. **彈性極高**：如果 JSON 漏傳了某些欄位，Builder 會直接賦予預設值 (null 或 0)，不會像全參數建構子那樣容易出錯。
    
3. **物件建立更流暢**：在程式碼中建立該物件時，可以使用 `VectorProfile.builder().fromYYR(1.0).build()`，可讀性極佳。
    

---

### 💡 總結：我該選哪一種？

|**情境**|**推薦寫法**|**說明**|
|---|---|---|
|**欄位少、全必填**|**解法一** (`@RequiredArgsConstructor`)|程式碼最乾淨，完全零 Jackson 侵入。需搭配 `ParameterNamesModule`。|
|**欄位多、有選填**|**解法二** (`@Builder` + `@Jacksonized`)|建構彈性最高，呼叫端可讀性好，且不挑 Jackson 環境。|

這兩種寫法都完美兼顧了 **「不可變性 (Immutability)」** 與 **「程式碼極簡化」**，可以依照專案團隊的習慣與物件的複雜度來選擇最適合的姿勢！

---

這份筆記是否符合您的需求？如果還需要調整任何排版或補充細節，請隨時告訴我！