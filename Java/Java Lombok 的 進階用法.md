
## 告別冗長的工具類呼叫：用 Lombok `@ExtensionMethod` 讓 Java 程式碼優雅變身

身為 Java 開發者，我們都寫過或看過這樣的程式碼：



```Java
String userInput = "  hello world  ";
if (!StringUtils.isBlank(userInput)) {
    String processedText = StringUtils.capitalize(userInput.trim());
    System.out.println(processedText);
}

int value = NumberUtils.toInt(someString, 0); // 失敗時返回預設值 0
```

這段程式碼完全沒問題，功能正常、邏輯清晰。`StringUtils` 和 `NumberUtils`。但你不禁會想：如果程式碼能讀起來更像自然語言，更符合物件導向的思維，那該有多好？

如果能像 C# 或 Kotlin 那樣，寫成這樣呢？

```Java
// 理想中的程式碼
String userInput = "  hello world  ";
if (!userInput.isBlank()) { // String 本身就有 isBlank()
    String processedText = userInput.trim().capitalize(); // 如果能鏈式呼叫就更棒了！
    System.out.println(processedText);
}

int value = someString.toInt(0);
```

這就是「擴充方法」(Extension Methods) 的魅力所在。它允許我們在不修改原始類別的情況下，為其「添加」新方法。雖然 Java 語言本身不直接支援此功能，但我們有好朋友 **Lombok**！透過它的 `@ExtensionMethod` 註解，我們幾乎可以完美地實現這個夢想。


#### 步驟 2：建立你的「擴充方法」容器

首先，建立一個普通的 `public` 類別，用來存放你想添加的靜態方法。這個類別就是擴充方法的「來源」。

**規則很簡單：**

1. 方法必須是 `public static`。
    
2. 方法的第一個參數，就是你要擴充的目標型別。
    

```Java
// src/main/java/com/myproject/extensions/StringExtensions.java

package com.myproject.extensions;

public class StringExtensions {

    public static String capitalize(String str) {
        if (str == null || str.isEmpty()) {
            return str;
        }
        return str.substring(0, 1).toUpperCase() + str.substring(1);
    }


    public static int toInt(String str, int defaultValue) {
        try {
            return Integer.parseInt(str);
        } catch (NumberFormatException e) {
            return defaultValue;
        }
    }
}
```

#### 步驟 3：在你的類別中使用 `@ExtensionMethod`

現在是最神奇的部分。在你想要使用這些擴充方法的類別上，加上 `@ExtensionMethod` 註解，並將剛剛建立的工具類傳入。



```Java
package com.myproject;

import com.myproject.extensions.StringExtensions; // 1. 匯入擴充方法類
import lombok.experimental.ExtensionMethod;     // 2. 匯入 Lombok 註解

// 3. 在類別上標註，Lombok 就會知道去哪裡找擴充方法
@ExtensionMethod({StringExtensions.class})
public class Application {

    public static void main(String[] args) {
        String greeting = "hello lombok!";
        
        // 見證奇蹟！直接在 String 物件上呼叫 capitalize()
        // Lombok 在編譯時會將它轉換成: StringExtensions.capitalize(greeting)
        String capitalizedGreeting = greeting.capitalize();
        
        System.out.println(capitalizedGreeting); // 輸出: Hello lombok!

        String numStr1 = "123";
        String numStr2 = "abc";

        // 同樣，直接呼叫 toInt()
        int number1 = numStr1.toInt(0);
        int number2 = numStr2.toInt(-1);
        
        System.out.println("Number 1: " + number1); // 輸出: Number 1: 123
        System.out.println("Number 2: " + number2); // 輸出: Number 2: -1

        // 鏈式呼叫的魅力
        String messyText = "  first word  ";
        String cleanText = messyText.trim().capitalize();
        System.out.println(cleanText); // 輸出: First word
    }
}
```

### 優點與注意事項

#### 優點：

1. **程式碼流暢度 (Fluent API)**: 程式碼的可讀性大大提升，特別是在進行一系列操作時，鏈式呼叫 `object.doA().doB().doC()` 遠比層層嵌套的 `HelperC.doC(HelperB.doB(HelperA.doA(object)))` 來得優雅。
    
2. **擴充 Final 類別**: 這是它相較於繼承的最大優勢。你可以為 `final` 類別（如 `String`, `Integer`）或任何你無法修改原始碼的第三方類別添加功能。
    
3. **關注點分離**: 業務邏輯和通用的工具函式可以被清晰地分開，但使用時又無縫地結合在一起。
    

