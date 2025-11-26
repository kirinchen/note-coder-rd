

# [React + TypeScript] Hook 入門：useState 與 useEffect 的型別魔法

Hooks 是 React 16.8 之後的核心，讓我們在 Function Component 裡面也能擁有「狀態 (State)」和處理「生命週期 (Lifecycle)」。加上 TypeScript 後，Hooks 變得非常強大，因為我們可以精確控制狀態的型別。

我們從最核心的兩個 Hook 開始：`useState` 和 `useEffect`。

## 1. `useState`：管理元件的記憶

`useState` 讓你的元件「記住」某些資訊（例如使用者輸入、開關狀態、API 資料）。

### 基礎用法：型別推論 (Type Inference)

對於簡單的型別（數字、字串、布林值），TypeScript 通常聰明到不需要你寫型別，它會自動推斷。



```TypeScript
import { useState } from 'react';

export const Counter = () => {
  // TS 自動推斷 count 是 number，因為初始值是 0
  const [count, setCount] = useState(0); 

  return (
    <button onClick={() => setCount(count + 1)}>
      Count is: {count}
    </button>
  );
};
```

### 進階用法：泛型 (Generics) 與 Null 處理

這是 TS 開發者最常用的模式。當狀態**一開始是空的 (null)**，但後來會有資料（例如從 API 抓取使用者資料），我們必須使用 `<Type | null>` 來明確告知 TS。



```TypeScript
interface User {
  id: number;
  name: string;
  email: string;
}

export const UserProfile = () => {
  // ⭐️ 關鍵：使用 <User | null> 泛型
  // 告訴 TS：這個 user 一開始是 null，但未來會是 User 型別
  const [user, setUser] = useState<User | null>(null);

  const login = () => {
    // 模擬登入，寫入資料
    setUser({ id: 1, name: "Gemini", email: "ai@google.com" });
  };

  return (
    <div>
      {/* TS 知道 user 可能是 null，所以強制你使用 Optional Chaining (?.) */}
      <h1>Welcome, {user?.name || "Guest"}</h1>
      
      {!user && <button onClick={login}>Login</button>}
    </div>
  );
};
```

> **為什麼這很重要？** 如果沒有寫 `<User | null>`，TS 會以為 `user` 的型別永遠是 `null`，當你試圖 `setUser({...})` 時就會報錯。

---

## 2. `useEffect`：處理副作用 (Side Effects)

`useEffect` 負責處理「渲染畫面之外」的事情，例如：打 API、訂閱事件、手動修改 DOM。

它的結構是：`useEffect(setupFunction, dependencyArray)`。

### 場景一：元件掛載時執行一次 (Mount)

通常用於 API 呼叫。依賴陣列 `[]` 代表「不依賴任何變數，只在出生時執行一次」。



```TypeScript
import { useEffect, useState } from 'react';

export const Clock = () => {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    // 1. 設定 Timer
    const timerId = setInterval(() => {
      setTime(new Date());
    }, 1000);

    // 2. Cleanup Function (清除函式)
    // 當元件被移除 (Unmount) 時，React 會執行這段，避免記憶體洩漏
    return () => {
      clearInterval(timerId);
      console.log('Clock stopped');
    };
  }, []); // 空陣列 = 只執行一次

  return <div>Current Time: {time.toLocaleTimeString()}</div>;
};
```

### 場景二：監聽特定變數變化 (Update)

當依賴陣列中的變數改變時，Effect 就會重新執行。



```TypeScript
interface SearchProps {
  keyword: string;
}

export const SearchResults = ({ keyword }: SearchProps) => {
  useEffect(() => {
    // 只有當 keyword 改變時，才會觸發搜尋
    if (keyword) {
      console.log(`Searching for: ${keyword}...`);
      // fetchApi(keyword)...
    }
  }, [keyword]); // ⭐️ 依賴陣列放入 keyword

  return <p>Searching results for {keyword}</p>;
};
```

---

## 3. 自定義 Hook (Custom Hook)：邏輯共用

這是 React 最優雅的設計。如果你發現有些邏輯在很多元件都用到（例如：偵測視窗寬度、切換開關），你可以把它抽成一個函數。

**命名慣例：一定要用 `use` 開頭。**

### 範例：`useToggle`

這是一個非常實用的 Hook，用來處理 Modal 開關、選單顯示等 Boolean 切換。



```TypeScript
// hooks/useToggle.ts
import { useState, useCallback } from 'react';

// 回傳一個 Tuple: [狀態, 切換函數]
export const useToggle = (initialState: boolean = false) => {
  const [state, setState] = useState<boolean>(initialState);

  // 使用 useCallback 避免函數重複建立 (優化效能)
  const toggle = useCallback(() => {
    setState((prev) => !prev);
  }, []);

  // 回傳 state 和 toggle 函數
  return [state, toggle] as const;
};
```

### 如何使用它？



```TypeScript
import { useToggle } from './hooks/useToggle';

export const ModalComponent = () => {
  // 就像使用 useState 一樣簡單，但邏輯被封裝了
  const [isOpen, toggleOpen] = useToggle(false);

  return (
    <div>
      <button onClick={toggleOpen}>
        {isOpen ? "Close Modal" : "Open Modal"}
      </button>
      
      {isOpen && <div className="modal">我是 Modal 內容...</div>}
    </div>
  );
};
```

---

## 總結 Cheat Sheet

|**Hook**|**用途**|**TypeScript 重點**|**常用寫法**|
|---|---|---|---|
|**useState**|記憶狀態|處理 Null 或複雜物件時需用泛型|`useState<User \| null>(null)`|
|**useEffect**|副作用 (API, Event)|無特殊泛型，注意依賴陣列 `[]`|`useEffect(() => { return cleanup }, [deps])`|
|**Custom Hook**|封裝共用邏輯|需明確定義回傳值的型別 (as const)|`const useMyHook = () => { ... }`|

---

### 下一步你可以做什麼？

現在你已經掌握了 React 的兩大支柱。

在真實的專案（特別是您提到的 Dashboad 或系統開發）中，我們經常遇到**「狀態需要在很多不同層級的元件中共用」**的情況（例如：使用者的登入資訊、全域的主題設定）。

如果您有興趣，我可以接著介紹 **`useContext`**，這是在不使用 Redux 等外部套件的情況下，管理全域狀態的原生解法。