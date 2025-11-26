

# [React + TypeScript] 掌握資料渲染：型別安全的 List 與 Dict 處理指南

在 React 開發中，處理 **List (Array)** 和 **Dictionary (Object)** 是基本功。而在 **TypeScript** 的加持下，我們可以明確定義這些資料的「形狀 (Shape)」，讓 IDE 告訴我們屬性是否存在，而不是等到瀏覽器報錯才發現拼錯字。

這篇文章將帶你了解如何在 TS 環境下優雅地渲染資料。

---

## 1. 處理 List (Array)：明確定義陣列內容

在 TypeScript 中，我們習慣先告訴元件這個陣列裡裝的是什麼（字串？數字？還是物件？）。

### 基礎範例：字串陣列 (`string[]`)



```TypeScript
import React from 'react';

// 定義資料型別：這是一個字串陣列
const fruits: string[] = ["Apple", "Banana", "Cherry", "Durian"];

export const FruitList: React.FC = () => {
  return (
    <div className="p-4 border rounded">
      <h3>Fruit List</h3>
      <ul>
        {/* map 的用法不變，但現在 TS 知道 'fruit' 是一個 string */}
        {fruits.map((fruit, index) => (
          <li key={index}>{fruit}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

## 2. 處理 Dictionary (Object)：Interface 是好朋友

TypeScript 最強大的地方在於 `interface` 或 `type`。當你定義好物件結構後，如果你試圖讀取一個不存在的屬性（例如把 `role` 拼成 `roll`），TS 會直接在編輯器畫紅線警告你。

### 步驟一：定義 Interface



```TypeScript
// 1. 定義物件的形狀
interface UserProfile {
  name: string;
  role: string;
  level: number;
  isActive: boolean;
}

const currentUser: UserProfile = {
  name: "Gemini",
  role: "AI Assistant",
  level: 99,
  isActive: true
};
```

### 步驟二：渲染 (與 Props 結合)



```TypeScript
// 定義 Component 的 Props 型別
interface UserCardProps {
  user: UserProfile;
}

export const UserCard: React.FC<UserCardProps> = ({ user }) => {
  return (
    <div className="card">
      {/* TS 會提示你有 name, role, level 這些屬性可用 */}
      <h2>{user.name}</h2>
      <p>Role: {user.role}</p>
      <p>Status: {user.isActive ? "Online" : "Offline"}</p>
    </div>
  );
};
```

### 進階：動態遍歷物件 (`Record` Type)

如果你要用 `Object.entries` 遍歷一個物件，通常適用於該物件屬性不固定，但類型統一的情況（例如設定檔）。



```TypeScript
// Record<KeyType, ValueType> 
// 這裡表示 Key 是字串，Value 可能是 string 或 boolean
const settings: Record<string, string | boolean> = {
  theme: "Dark",
  notifications: true,
  privacy: "Public"
};

export const SettingsList: React.FC = () => {
  return (
    <dl>
      {Object.entries(settings).map(([key, value]) => (
        <div key={key}>
          <dt>{key}:</dt>
          {/* 因為 value 可能是 boolean，React 不會渲染 boolean，需轉字串 */}
          <dd>{value.toString()}</dd>
        </div>
      ))}
    </dl>
  );
};
```

---

## 3. 實戰場景：List of Objects (物件陣列)

這是最常見的 API 回傳格式。我們通常會定義一個 `interface` 來描述單筆資料。



```TypeScript
// 1. 定義單篇文章的資料結構
interface Post {
  id: number;
  title: string;
  content: string;
}

const posts: Post[] = [
  { id: 101, title: "React 101", content: "Learn the basics..." },
  { id: 102, title: "Typescript Tips", content: "Interfaces are cool..." },
];
```

### 最佳實踐：拆分元件並定義 Props

這展示了 TS 如何保護元件的邊界。如果父元件傳入錯誤的 props，程式碼根本編譯不過。



```TypeScript
// 2. 子元件：定義它需要什麼 Props
// 這裡可以直接使用 Post interface，或者用 Pick 取出需要的欄位
interface PostItemProps {
  title: string;
  content: string;
}

const PostItem: React.FC<PostItemProps> = ({ title, content }) => (
  <div style={{ marginBottom: '20px' }}>
    <h3>{title}</h3>
    <p>{content}</p>
  </div>
);

// 3. 主列表元件
export const BlogList: React.FC = () => {
  return (
    <section>
      <h1>Latest Posts</h1>
      
      {posts.map((post) => (
        <PostItem 
          key={post.id} 
          title={post.title}
          content={post.content}
          // 如果這裡少傳了 content，TS 會立刻報錯
        />
      ))}
    </section>
  );
};
```

---

## TypeScript 重點筆記

|**概念**|**寫法**|**為什麼要這樣寫？**|
|---|---|---|
|**定義資料**|`interface User { name: string }`|讓編輯器知道 `user.` 後面可以接什麼。|
|**陣列型別**|`User[]` 或 `Array<User>`|確保你不會把 `string` 塞進 `number` 陣列。|
|**Component**|`React.FC<PropsType>`|定義元件接收的 Props 格式，防止傳錯參數。|
|**Map 參數**|`posts.map((post) => ...)`|因為 `posts` 已經定義為 `Post[]`，這裡的 `post` 會自動推斷為 `Post` 型別，不需重寫。|

---

### 下一步你可以做什麼？

既然你已經使用了 TypeScript，通常下一步我們會遇到「**非同步資料的型別定義**」。

如果您有興趣，我可以示範**如何定義 API 回傳結果的 Interface**，並使用 `useEffect` 抓取資料後，將這些 JSON 資料安全地放入 TS 的狀態 (`useState<Post[]>`) 中。這樣能完整模擬真實開發流程。