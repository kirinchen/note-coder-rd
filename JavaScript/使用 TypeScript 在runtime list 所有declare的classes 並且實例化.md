---
title: 使用 TypeScript 在runtime list 所有declare的classes 並且實例化

---

# 使用 TypeScript 在runtime list 所有declare的classes 並且實例化
標題：使用 TypeScript 動態實例化子類別

在 TypeScript 中，有時候我們想要在運行時動態地實例化特定的子類別。雖然 TypeScript 沒有像 Java 那樣的反射機制，但我們可以使用第三方庫 `reflect-metadata` 來實現類似的功能。讓我們來看一個示例，如何使用 `reflect-metadata` 在運行時動態地實例化特定的子類別。

首先，我們需要安裝 `reflect-metadata` 库：

```bash
npm install reflect-metadata
```

然後，讓我們來編寫 TypeScript 代碼：

```typescript
import "reflect-metadata";
const _componentList = new Array<any>();
// 定義一個裝飾器，將類標記為組件
const Component = Symbol("Component");
function ComponentDecorator(target: any) {
    Reflect.defineMetadata(Component, true, target);
    _componentList.push(target);
}

// 根類
class Root {}

// 子類別 A
@ComponentDecorator
class SubclassA extends Root {
    constructor() {
        super();
        console.log("正在創建 SubclassA 的實例");
    }
}

// 子類別 B
class SubclassB extends Root {}

// 子類別 C
@ComponentDecorator
class SubclassC extends Root {
    constructor() {
        super();
        console.log("正在創建 SubclassC 的實例");
    }
}




// 使用方法
const subclasses = _componentList;
console.log(subclasses); // 輸出：[SubclassA, SubclassC]

// 動態實例化 SubclassA 和 SubclassC
const instances = subclasses.map(Subclass => new Subclass());
```

在這個代碼中，`instances` 數組將包含 `SubclassA` 和 `SubclassC` 的實例，因為這些是帶有 `Component` 裝飾器的子類別。每個子類別的構造函數將被調用，您將看到創建實例的日誌消息。這個技巧可以讓您動態地根據需要創建特定的子類別實例，而不需要在代碼中直接引用它們。