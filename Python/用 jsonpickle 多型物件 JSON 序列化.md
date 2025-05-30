

# 🐍 Python 精簡教學：用 `jsonpickle` 儲存與還原多型物件！

在 Python 中，我們常需要把自訂物件轉成 JSON 儲存或傳輸。不過標準的 `json` 模組只能處理 dict、list、str、int 等基本型別，遇到類別（尤其是多型）時就 GG 😵。

這時候，**`jsonpickle` 就是你的好幫手**！

---

## ✅ `jsonpickle` 是什麼？

`jsonpickle` 是一個 Python 套件，可以將 **任意 Python 物件（包含類別、巢狀、多型）序列化成 JSON**，並支援還原。

它的效果很像 Java 的 `@JsonTypeInfo(use = CLASS)` 或 Python 的 `pickle`，但是 JSON 格式，更通用！

---

## 🚀 快速上手：一分鐘搞懂 `jsonpickle`

### ✨ 安裝

```bash
pip install jsonpickle
```

---

### 🐶 假設我們有以下類別結構：

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, bark=True):
        super().__init__(name)
        self.bark = bark
```

---

### 🔁 序列化 + 反序列化

```python
import jsonpickle

dog = Dog("Buddy")

# 序列化成 JSON 字串
json_str = jsonpickle.encode(dog)
print(json_str)

# 反序列化回原本的 Dog 物件
obj = jsonpickle.decode(json_str)
print(type(obj), obj.__dict__)
```

---

### 📦 輸出 JSON 範例

```json
{
  "py/object": "__main__.Dog",
  "name": "Buddy",
  "bark": true
}
```

這裡的 `"py/object"` 就是類似 Java 的 `"@class"`，自動包含完整類型資訊，還原時會用 `import` 方式動態載入類別。

---

## 🧠 使用情境

|使用場景|是否適合|
|---|---|
|儲存/還原動態型別資料|✅ 非常適合|
|序列化巢狀資料或循環引用|✅ 處理得很好|
|跨語言（送 JSON 給前端）|❌ 不推薦，格式太特殊|
|高安全性場景（接受用戶輸入）|⚠️ 謹慎使用，勿 decode 不可信 JSON|

---

## 🛡️ 安全提醒！

因為 `jsonpickle.decode()` 會依照 `py/object` 動態建立類別實例，**千萬不要用來反序列化不可信來源的 JSON**！

如需與外部系統互動，建議使用 `dict()` + `json` 手動序列化或用 `pydantic`。

---

## 🏁 結語

|你要的功能|用 `jsonpickle` ✅|
|---|---|
|序列化/反序列化自訂類別|✅|
|保留類型資訊|✅ 自動加 `"py/object"`|
|快速還原 Python 多型物件|✅ 一行搞定|

簡單一句話：**jsonpickle 是 Python 版的 `@JsonTypeInfo(use = CLASS)`，超適合處理多型、巢狀與類別封裝物件！**
