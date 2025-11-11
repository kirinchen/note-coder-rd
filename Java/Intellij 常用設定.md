---
title: Intellij 常用設定
tags: [java, IDE, Intellij, TODO]

---

# Intellij 常用設定

## 快捷鍵

### 選擇文字大小寫
```shell
ctrl + shift + u
```

### 選擇文字駝峰化
有一個功能極其強大且幾乎是必裝的插件叫做 **String Manipulation**，它完美地解決了這個問題，並且提供了數十種其他的字串處理功能。


#### 1. 安裝插件

1. 打開 IntelliJ IDEA 的設定 (Settings/Preferences)。
    
    - Windows/Linux: `File` -> `Settings`
        
    - macOS: `IntelliJ IDEA` -> `Preferences`
        
2. 在左側選擇 `Plugins`。
    
3. 確定你在 `Marketplace` 分頁。
    
4. 搜尋 "**String Manipulation**"。
    
5. 找到該插件後，點擊 `Install` 並依照指示重啟 IDE。
    

#### 2. 如何使用

安裝完成後，你就可以輕鬆轉換任何選擇的文字了。

1. **選取**你想要轉換的文字 (例如 `hello_world` 或 `Hello World`)。
    
2. 按下快捷鍵 **`Alt + M`** (Windows/Linux) 或 **`⌥ + M`** (macOS)。
    
3. 這會彈出一個操作選單，在 `Switch Case` 的子選單中，你可以找到各種命名風格的轉換選項。
    
4. 選擇 `to camelCase` 即可。
    

**常見的轉換選項包括：**

- `to camelCase` (例如：`helloWorld`)
    
- `to snake_case` (例如：`hello_world`)
    
- `to kebab-case` (例如：`hello-world`)
    
- `to PascalCase` (例如：`HelloWorld`)
    
- `to SCREAMING_SNAKE_CASE` (例如：`HELLO_WORLD`)
    
- ...以及更多其他實用的功能。
    

你甚至可以為 `to camelCase` 這個特定操作綁定一個專屬的快捷鍵，這樣就不用每次都從選單中選擇了。


### 替代方法：重構 (Refactor) 功能

如果你要修改的是一個已經存在的**變數、方法或類別名稱**，你可以使用 IntelliJ 強大的重構功能。

1. 將你的滑鼠游標放在該變數/方法/類別名稱上。
    
2. 按下 **`Shift + F6`** (Rename)。
    
3. 在彈出的輸入框中，直接修改成你想要的駝峰式命名。
    
4. 按下 `Enter`。IntelliJ 會自動幫你更新所有使用到這個名稱的地方，非常安全。
    

這種方法適用於程式碼中的符號重命名，但如果你只是想轉換一段普通的文字（例如在註解或字串中），那麼 **String Manipulation** 插件是最好的選擇。

總結來說，**安裝 String Manipulation 插件** 是解決這個問題最完美的方法。

###  Code completion vs Cyclic expand word
* Code completion : 會有下拉選單讓你選建議
* Cyclic expand word : 自動幫你補完後面的字(通常都不是....)

## JUnit Run / Debug

## 設定啟動參數

### Spring boot profile

![](https://i.imgur.com/2Tdh75D.png)


```shell=
spring.profiles.active=dev
```






###### tags: `java` `IDE` `Intellij` `TODO`