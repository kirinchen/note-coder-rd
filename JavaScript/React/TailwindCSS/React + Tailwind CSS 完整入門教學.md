

### 1. 環境建置 (TypeScript 版)

我們在建立 Vite 專案時，需要指定 TypeScript 的模板 (`react-ts`)。

開啟終端機，依序執行：

**步驟 A: 建立 TypeScript 專案**



```Bash
# 1. 建立專案 (注意最後面的 --template react-ts)
npm create vite@latest my-app -- --template react-ts

# 2. 進入資料夾
cd my-app

# 3. 安裝依賴
npm install
```

**步驟 B: 安裝 Tailwind v4 與 Vite 插件**

指令與之前相同：



```Bash
npm install tailwindcss @tailwindcss/vite
```

**步驟 C: 修改 `vite.config.ts`**

注意副檔名現在是 `.ts`。請將 Tailwind 插件加入設定中：



```TypeScript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite' // 1. 引入插件

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    tailwindcss(), // 2. 啟用插件
  ],
})
```

**步驟 D: 設定 CSS 入口**

修改 `src/index.css`，清空內容並加入：



```CSS
@import "tailwindcss";
```

---

### 2. 打造基本 Layout (加入型別定義)

在 TypeScript 中，最常見的錯誤就是「沒有定義 props 的型別」。對於 Layout 元件，我們必須告訴 TS `children` 是什麼。

在 `src` 下建立 `components` 資料夾，新增 `Layout.tsx` (注意是 .tsx)。

**檔案：`src/components/Layout.tsx`**



```TypeScript
import React, { ReactNode } from 'react';

// 定義 Props 的介面
interface LayoutProps {
  children: ReactNode; // ReactNode 包含了 JSX 元素、字串、數字等所有可渲染內容
}

const Layout = ({ children }: LayoutProps) => {
  return (
    <div className="flex flex-col min-h-screen bg-gray-50 text-gray-800">
      {/* Header */}
      <header className="sticky top-0 z-50 bg-white shadow-sm border-b border-gray-200">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
          <h1 className="text-xl font-bold text-indigo-600 tracking-wider">MyReactApp (TS)</h1>
          <nav className="space-x-4 text-sm font-medium">
            <a href="#" className="hover:text-indigo-600 transition-colors">首頁</a>
            <a href="#" className="hover:text-indigo-600 transition-colors">關於</a>
          </nav>
        </div>
      </header>

      {/* Main Content */}
      <main className="flex-grow w-full max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {children}
      </main>

      {/* Footer */}
      <footer className="bg-white border-t border-gray-200 py-6 text-center text-gray-500 text-sm">
        &copy; {new Date().getFullYear()} React + TypeScript + Tailwind v4.
      </footer>
    </div>
  );
};

export default Layout;
```

---

### 3. 建立互動元件 (TypeScript State)

在 `CounterCard.tsx` 中，雖然 TS 通常能自動推斷 `useState(0)` 是數字，但明確寫出泛型 `<number>` 是個好習慣。

**檔案：`src/components/CounterCard.tsx`**



```TypeScript
import React, { useState } from 'react';

const CounterCard = () => {
  // 明確指定 state 為 number 型別
  const [count, setCount] = useState<number>(0);

  return (
    <div className="bg-white rounded-xl shadow-lg border border-gray-100 overflow-hidden hover:shadow-2xl transition-shadow duration-300">
      <div className="p-6">
        <div className="uppercase tracking-wide text-sm text-indigo-500 font-semibold">
          互動測試 (TS)
        </div>
        <p className="block mt-1 text-lg leading-tight font-medium text-black">
          TypeScript 計數器
        </p>
        <p className="mt-2 text-gray-500">
          點擊次數：
          <span className="font-bold text-indigo-600 text-2xl ml-2">{count}</span>
        </p>
        
        <div className="mt-6 flex space-x-3">
          <button 
            onClick={() => setCount(count + 1)}
            className="px-4 py-2 bg-indigo-600 text-white text-sm font-medium rounded-md hover:bg-indigo-700 active:bg-indigo-800 transition-colors"
          >
            增加 (+1)
          </button>
          
          <button 
            onClick={() => setCount(0)}
            className="px-4 py-2 bg-gray-100 text-gray-700 text-sm font-medium rounded-md hover:bg-gray-200 transition-colors"
          >
            重置
          </button>
        </div>
      </div>
    </div>
  );
};

export default CounterCard;
```

---

### 4. 組合所有內容 (App.tsx)

最後修改 `src/App.tsx`。程式碼邏輯與 JS 版相同，但現在我們享有完整的型別檢查保護。

**檔案：`src/App.tsx`**



```TypeScript
import Layout from './components/Layout';
import CounterCard from './components/CounterCard';

function App() {
  return (
    <Layout>
      <div className="space-y-8">
        {/* 標題區 */}
        <div className="text-center">
          <h2 className="text-3xl font-extrabold text-gray-900 sm:text-4xl">
            React + TypeScript + Tailwind
          </h2>
          <p className="mt-4 text-lg text-gray-500">
            使用 Vite 建置的現代化前端開發環境。
          </p>
        </div>

        {/* Grid 排版應用：手機單欄，平板以上雙欄 */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          
          {/* 元件 1 */}
          <CounterCard />

          {/* 元件 2：靜態展示 */}
          <div className="bg-indigo-50 rounded-xl p-6 border border-indigo-100 flex flex-col justify-center items-center text-center">
            <h3 className="text-lg font-bold text-indigo-800 mb-2">Type Safe!</h3>
            <p className="text-indigo-700 mb-4">
              使用 TypeScript 可以避免大多數常見的 Props 傳遞錯誤。
            </p>
            <div className="flex gap-2">
               <span className="px-3 py-1 bg-blue-100 text-blue-700 text-xs font-bold rounded-full">
                 TypeScript
               </span>
               <span className="px-3 py-1 bg-cyan-100 text-cyan-700 text-xs font-bold rounded-full">
                 Tailwind v4
               </span>
            </div>
          </div>

        </div>
      </div>
    </Layout>
  );
}

export default App;
```

---

### 5. 啟動與驗證

現在執行專案：



```Bash
npm run dev
```

TypeScript 小撇步：

在使用 Tailwind 時，如果你將滑鼠游標移到 className 上（例如 bg-indigo-500），如果你的 VS Code 有安裝 "Tailwind CSS IntelliSense" 擴充套件，它會顯示該 class 對應的實際 CSS 數值（例如 background-color: rgb(99 102 241);）。在 TS 環境下，這個功能依然運作完美。

## Tailwind CSS Grid 系統
這是一份關於 **Tailwind CSS Grid 系統** 的完整說明。掌握這裡的概念，您幾乎可以處理 90% 的網頁排版需求。

---

### 1. Grid 的基本語法結構

在 Tailwind 中，啟動 Grid 佈局只需要三個關鍵屬性：

1. **容器宣告 (`grid`)**：告訴瀏覽器「這裡要用網格排版」。
    
2. **切分欄數 (`grid-cols-{n}`)**：定義「直的要切成幾份」。
    
3. **設定間距 (`gap-{n}`)**：定義格子之間的距離（水溝）。
    

#### 語法範例



```JavaScript
<div className="grid grid-cols-3 gap-4">
  {/* 這裡面的子元素會自動變成 3 欄排列 */}
  <div>1</div>
  <div>2</div>
  <div>3</div>
  <div>4 (會自動換行到第二列)</div>
</div>
```

---

### 2. 子元素的控制 (`col-span`)

定義好父容器切成幾份後，子元素預設都是 佔 1 份。

如果您希望某個區塊比較寬，就要使用 col-span（跨欄）。

|**寫法**|**意義**|**常見用途**|
|---|---|---|
|**`col-span-1`**|(預設值) 佔據 1 格|一般卡片、列表項目|
|**`col-span-2`**|跨越 2 格|重點卡片、較寬的圖片|
|**`col-span-full`**|跨越 **整行** (無論總共有幾欄)|標題 (Header)、分隔線、Banner|

---

### 3. 常用解析度 (Breakpoints) 與寫法

Tailwind 採用 **「手機優先 (Mobile First)」** 的邏輯。這意味著：

1. **不加前綴的 class** (如 `grid-cols-1`) 是寫給 **手機** 看的。
    
2. **`md:`** 是寫給 **平板** 以上看的。
    
3. **`lg:`** 是寫給 **筆電/桌機** 以上看的。
    

以下是業界最常用的 **「響應式卡片列表」** 寫法清單：

#### 常用斷點表 (Breakpoints Table)

|**前綴代號**|**觸發寬度 (min-width)**|**適用裝置範例**|
|---|---|---|
|**(無)**|0px ~|**手機直向** (iPhone SE, Pixel)|
|**`sm:`**|640px ~|**手機橫向** / 大尺寸手機|
|**`md:`**|768px ~|**平板直向** (iPad mini, iPad)|
|**`lg:`**|1024px ~|**平板橫向 / 小筆電** (iPad Pro, Laptop 13")|
|**`xl:`**|1280px ~|**標準桌機螢幕** (Desktop 24")|
|**`2xl:`**|1536px ~|**超大螢幕** (Ultrawide)|

---

### 4. 實戰範例：完美的響應式列表 (The Golden Pattern)

這是最經典的寫法：**手機單欄 -> 平板雙欄 -> 筆電三欄 -> 桌機四欄**。

請將這段 Code 貼入您的專案中感受一下：



```TypeScript
import React from 'react';

const ResponsiveGrid = () => {
  // 產生 8 個假資料來測試排版
  const items = Array.from({ length: 8 }, (_, i) => i + 1);

  return (
    <div className="p-8 bg-gray-50 min-h-screen">
      <h2 className="text-2xl font-bold mb-6 text-center">響應式 Grid 展示</h2>
      
      {/* 關鍵寫法解析：
        1. grid: 啟動網格
        2. gap-6: 格子間距 24px
        3. grid-cols-1: 手機版 (預設) 1 欄
        4. sm:grid-cols-2: 寬度 > 640px 時，變 2 欄
        5. lg:grid-cols-3: 寬度 > 1024px 時，變 3 欄
        6. xl:grid-cols-4: 寬度 > 1280px 時，變 4 欄
      */}
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        
        {items.map((num) => (
          <div key={num} className="bg-white p-6 rounded-xl shadow-sm border border-gray-200 hover:shadow-md transition-shadow">
            <div className="text-indigo-500 font-bold text-xl mb-2">Item {num}</div>
            <p className="text-gray-600 text-sm">
              縮放視窗寬度，你會看到這個卡片的排列方式自動改變。
            </p>
            <div className="mt-4 text-xs font-mono text-gray-400 bg-gray-100 p-2 rounded">
              {/* 這裡用來顯示目前的 Grid 設定 (示意用) */}
              col-span-1
            </div>
          </div>
        ))}

      </div>
    </div>
  );
};

export default ResponsiveGrid;
```

### 5. 特殊排版技巧 (進階)

有時候我們不想要均分，而是想要 **「左邊窄、右邊寬」** (例如 Sidebar 佈局)。Tailwind 允許你使用 `[]` (Arbitrary values) 來寫自定義比例。



```TypeScript
{/* 定義兩欄：左邊固定 250px，右邊佔滿剩餘空間 (1fr) */}
<div className="grid grid-cols-[250px_1fr] gap-4 min-h-screen">
  
  {/* 左側邊欄 */}
  <div className="bg-gray-800 text-white p-4">
    Sidebar (固定 250px)
  </div>

  {/* 右側內容 */}
  <div className="bg-white p-4">
    Main Content (自動填滿)
  </div>

</div>
```

### 總結記憶點

1. **手機優先**：永遠先寫 `grid-cols-1`，再用 `md:` 加欄位。
    
2. **Gap 很好用**：不要用 margin (`m-4`) 來推開格子，用 `gap-4` 最乾淨。
    
3. **標準口訣**：`grid-cols-1 md:grid-cols-2 lg:grid-cols-3` (最萬用的 RWD 寫法)。
    



## Utility Classes (工具類別)

這些代碼是 Tailwind CSS 的靈魂，稱為 **Utility Classes (工具類別)**。每一個類別都對應到一行標準的 CSS 樣式。

這串組合通常是用來設計一個 **「次要按鈕」** 或 **「標籤」** 的樣式。

我們可以把它們拆解成 **5 大類別** 來理解：

### 1. 間距 (Spacing)

Tailwind 的數字通常以 **4px** 為一個單位（1單位 = 0.25rem）。

|**類別 (Class)**|**對應 CSS**|**白話文解釋**|
|---|---|---|
|**`mt-2`**|`margin-top: 0.5rem;`|**上方**留白 8px (2 x 4px)。|
|**`px-4`**|`padding-left: 1rem;`<br><br>  <br><br>`padding-right: 1rem;`|**左右**內距 16px (4 x 4px)。讓按鈕看起來寬一點。|
|**`py-2`**|`padding-top: 0.5rem;`<br><br>  <br><br>`padding-bottom: 0.5rem;`|**上下**內距 8px。控制按鈕的高度感。|

> **💡 記憶法：**
> 
> - `m` = margin (外距), `p` = padding (內距)
>     
> - `t` = top, `b` = bottom, `l` = left, `r` = right
>     
> - `x` = x軸 (左右), `y` = y軸 (上下)
>     

### 2. 顏色 (Colors)

Tailwind 的顏色由「色名」+「深淺度 (50-950)」組成。數字越大越深。

|**類別 (Class)**|**效果預覽**|**白話文解釋**|
|---|---|---|
|**`bg-gray-100`**|⬜ (淺灰底)|背景色設為很淺的灰色。|
|**`text-gray-700`**|⬛ (深灰字)|文字顏色設為深灰色（比純黑 `black` 柔和一點）。|

### 3. 文字與字型 (Typography)

|**類別 (Class)**|**對應 CSS**|**白話文解釋**|
|---|---|---|
|**`text-sm`**|`font-size: 0.875rem;`|**小號字體** (約 14px)。標準網頁字通常是 `text-base` (16px)。|
|**`font-medium`**|`font-weight: 500;`|**中等粗細**。比一般 (`normal/400`) 粗一點，但不到粗體 (`bold/700`)。|

### 4. 形狀 (Borders)

|**類別 (Class)**|**對應 CSS**|**白話文解釋**|
|---|---|---|
|**`rounded-md`**|`border-radius: 0.375rem;`|**中等圓角** (6px)。讓按鈕的四個角不要那麼尖銳，看起來比較現代。|

### 5. 互動與動畫 (Interactivity)

這是讓按鈕「活起來」的關鍵。

|**類別 (Class)**|**功能**|**白話文解釋**|
|---|---|---|
|**`hover:bg-gray-200`**|滑鼠懸停效果|當滑鼠 **移上去 (Hover)** 時，背景色會變成 `gray-200` (比原本的 100 深一點)，產生「被按壓」的視覺回饋。|
|**`transition-colors`**|過渡動畫|當顏色改變時（例如滑鼠移上去變色），不要瞬間切換，而是要有 **漸變效果**，讓視覺更滑順。|

---

### 總結效果

把這些合在一起，這行程式碼創造了一個：

「上方有留白、灰色背景、圓角、深灰色小字，且滑鼠移上去會優雅變深色的按鈕。」

這種寫法的好處是你不需要去 CSS 檔案寫類似 `.my-custom-button { ... }` 這麼長一串程式碼，直接寫在 HTML/JSX 裡就能完成設計。