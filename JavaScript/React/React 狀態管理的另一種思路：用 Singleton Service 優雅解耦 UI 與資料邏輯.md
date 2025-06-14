

## React 狀態管理的另一種思路：用 Singleton Service 優雅解耦 UI 與資料邏輯

在開發 React 應用時，狀態管理永遠是核心議題。當應用變得複雜，例如需要一個元件（如 Tree View）的選擇去連動另一個元件（如 List View）的內容時，我們常會陷入 `props` 逐層傳遞（Prop Drilling）的困境，或是需要引入像 Redux、Zustand 這樣功能強大的狀態管理庫。

然而，對於中等規模的應用，或是在想保持輕量、減少依賴的場景下，有沒有更簡潔、更優雅的作法？

答案是肯定的。透過結合兩種經典設計模式——**單例模式（Singleton）**與**觀察者模式（Observer）**——我們可以打造一個獨立於 React 框架的資料服務層，並透過自定義 Hook（Custom Hook）將它與 UI 元件無縫接軌。

本文將帶你一步步實現這個架構，讓你看到一個 UI 與邏輯徹底解耦的清爽世界。

---

### 架構設計：三大核心角色

這個模式主要由三個部分組成，各司其職，完美分工：

1. **Singleton Service (大腦 🧠)**
    
    - 這是一個純粹的 TypeScript/JavaScript 類別，負責管理應用的核心狀態資料。
    - **單例模式**確保整個應用中只有「一個」資料實例，成為唯一的「真實來源 (Single Source of Truth)」。
    - 它同時扮演「發布者」的角色，提供 `subscribe` 方法讓外部訂閱資料變更，並在資料異動時（如新增、刪除），透過 `notify` 方法通知所有「訂閱者」。
2. **Custom Hook (橋樑 🌉)**
    
    - 這是連接「非 React 世界」的 Service 與「React 世界」的 UI 元件之間的關鍵橋樑。
    - 它封裝了訂閱與取消訂閱的生命週期邏輯。在元件掛載時向 Service 註冊，在卸載時自動取消，完美避免了記憶體洩漏。
    - 當 Service 發出變更通知時，這個 Hook 會更新自身的 state，從而觸發使用它的 React 元件進行重新渲染（re-render）。
3. **React Components (演員 🎭)**
    
    - UI 元件變得非常「單純」。它們不再需要自己管理複雜的共享狀態。
    - 只需呼叫自定義 Hook，就能輕鬆取得最新的資料來進行渲染，以及取得 Service 的實例來呼叫其方法以觸發資料變更。

它們之間的關係如下：

`Service (管理資料) <==> Hook (訂閱與通知) <==> Component (渲染與觸發)`

---

### 實作細節：一步步打造我們的資料服務

現在，讓我們用程式碼來實現這個架構。

#### 第一步：打造資料核心 `TreeViewModelService`

這是我們整個架構的核心，它獨立於 React，只專注於資料處理。

`src/services/TreeViewModelService.ts`



``` TypeScript
import { TreeNode } from 'primereact/treenode';

// 定義 List View 中每個項目的型別
export interface ListItem {
  id: string;
  name: string;
}

// 擴充 PrimeReact 的 TreeNode，使其可以攜帶我們自訂的資料
export interface AppTreeNode extends TreeNode {
  data?: ListItem[]; // 每個節點對應的列表資料
  children?: AppTreeNode[];
}

class TreeViewModelService {
  // 1. Singleton 模式：靜態私有實例
  private static instance: TreeViewModelService;

  // 存放所有訂閱者的回呼函式
  private subscribers: Set<() => void> = new Set();

  // 我們的核心資料，私有化以保護它
  private treeData: AppTreeNode[] = [ /* ... 初始資料 ... */ ];

  // 2. Singleton 模式：私有化建構子，防止外部 new
  private constructor() {
    // 初始化資料可以放在這裡
    console.log("TreeViewModelService Initialized!");
  }

  // 3. Singleton 模式：提供靜態方法來取得唯一的實例
  public static getInstance(): TreeViewModelService {
    if (!TreeViewModelService.instance) {
      TreeViewModelService.instance = new TreeViewModelService();
    }
    return TreeViewModelService.instance;
  }

  // 4. Observer 模式：通知所有訂閱者
  private notify() {
    console.log("Notifying all subscribers of data change...");
    this.subscribers.forEach(callback => callback());
  }

  // --- 公開 API ---

  // 5. Observer 模式：提供訂閱方法
  public subscribe(callback: () => void): () => void {
    this.subscribers.add(callback);
    // 返回一個取消訂閱的函式，這對於在 React useEffect 中清理至關重要
    return () => {
      this.subscribers.delete(callback);
      console.log("A subscriber has unsubscribed.");
    };
  }

  // 取得整棵樹的資料
  public getTree(): AppTreeNode[] {
    return this.treeData;
  }

  // 根據節點 key 取得對應的 List 資料
  public getListForNode(key: string | null): ListItem[] | undefined {
    // ... 實作搜尋邏輯 ...
  }

  // 新增項目到指定節點的 List
  public addItemToNodeList(nodeKey: string, item: ListItem) {
    const list = this.getListForNode(nodeKey);
    if (list) {
      list.push(item);
      // **關鍵：修改資料後，立即廣播通知！**
      this.notify();
    }
  }

  // ... 其他資料操作方法，如 delete, update，都應在結束時呼叫 this.notify()
}

// 導出唯一的 Service 實例
export const treeViewModelService = TreeViewModelService.getInstance();
```

#### 第二步：建立 React 與 Service 的橋樑 `useTreeViewModel`

這個自定義 Hook 是魔法發生的地方。它讓我們的元件能夠「響應」Service 的變化。

`src/hooks/useTreeViewModel.ts`



```TypeScript
import { useState, useEffect } from 'react';
import { treeViewModelService } from '../services/TreeViewModelService';

export const useTreeViewModel = () => {
  // 這個 state 的唯一目的就是觸發重新渲染。它的值本身不重要。
  const [, setTick] = useState(0);

  useEffect(() => {
    // 元件掛載時，向 service 訂閱。
    const unsubscribe = treeViewModelService.subscribe(() => {
      // 當 service 呼叫 notify()，此回呼函式會被執行。
      // 我們透過更新 state 來強制元件重新渲染，以取得最新資料。
      setTick(tick => tick + 1);
    });

    // 元件卸載時，執行 subscribe 回傳的清理函式來取消訂閱。
    // 這一步至關重要，可以防止記憶體洩漏和不必要的錯誤。
    return unsubscribe;
  }, []); // 空依賴陣列確保此 effect 只在掛載和卸載時執行一次。

  // 將 service 實例回傳，讓元件可以直接使用它的公開方法。
  return { service: treeViewModelService };
};
```

#### 第三步：組裝我們的 UI 元件

現在，我們的元件可以保持極度的簡潔。

`src/components/RightList.tsx`



```TypeScript
import React from 'react';
import { Button } from 'primereact/button';
import { OrderList } from 'primereact/orderlist';
import { useTreeViewModel } from '../hooks/useTreeViewModel';
import { ListItem } from '../services/TreeViewModelService';

interface RightListProps {
  selectedNodeKey: string | null;
}

export const RightList: React.FC<RightListProps> = ({ selectedNodeKey }) => {
  // 1. 呼叫 Hook，取得 service 實例並建立訂閱關係。
  const { service } = useTreeViewModel();
  
  // 2. 直接從 service 取得渲染所需的最新資料。
  const listData = service.getListForNode(selectedNodeKey) || [];

  const handleAddItem = () => {
    if (selectedNodeKey) {
      const newItem: ListItem = {
        id: `new-${Date.now()}`,
        name: `新成員 ${Math.floor(Math.random() * 100)}`,
      };
      // 3. 呼叫 service 的方法來「請求」修改資料。
      // 元件本身不關心資料如何被修改，只負責發出指令。
      service.addItemToNodeList(selectedNodeKey, newItem);
    }
  };
  
  // ... JSX 渲染邏輯 ...
  return (
    <div>
      <Button label="新增項目" onClick={handleAddItem} disabled={!selectedNodeKey} />
      <OrderList value={listData} /* ... */ />
    </div>
  );
};
```

---

### 實際流程：一次完整的資料流動

讓我們串連起來，看看當使用者點擊「新增項目」按鈕時，發生了什麼：

1. **使用者點擊**: `RightList` 元件中的 `handleAddItem` 函式被呼叫。
2. **請求變更**: `handleAddItem` 呼叫 `service.addItemToNodeList()`。
3. **資料更新**: `TreeViewModelService` 實例更新其內部的 `treeData` 陣列。
4. **發出廣播**: Service 呼叫 `this.notify()`。
5. **接收通知**: 所有透過 `useTreeViewModel` Hook 訂閱的元件（`LeftTree` 和 `RightList`）的 `useEffect` 回呼函式被觸發。
6. **觸發渲染**: `setTick(tick + 1)` 被執行，React 偵測到 state 變化，安排 `LeftTree` 和 `RightList` 重新渲染。
7. **畫面更新**: 元件重新渲染時，再次從 `service` 取得最新的資料，畫面上成功顯示了新增的項目。整個過程流暢且自動。

---

### 這個模式的優勢

- **高度解耦 (Decoupling)**: UI 與業務邏輯徹底分離。你可以輕易更換 UI 庫，甚至更換整個前端框架，而核心的 Service 保持不變。
- **易於測試 (Testability)**: 你可以像測試普通 JavaScript 物件一樣，獨立對 `TreeViewModelService` 進行單元測試，無需瀏覽器環境，也無需處理 React 的複雜性。
    
    
    
    ```JavaScript
    // 使用 Jest 測試範例
    test('should add an item to a node list', () => {
      const service = TreeViewModelService.getInstance();
      const initialCount = service.getListForNode('0-0').length;
      service.addItemToNodeList('0-0', { id: 'test', name: 'Test Item' });
      const newCount = service.getListForNode('0-0').length;
      expect(newCount).toBe(initialCount + 1);
    });
    ```
    
- **單一真實來源 (Single Source of Truth)**: 所有狀態都由 Singleton Service 統一管理，杜絕了資料不一致的問題。
- **絕佳的擴展性 (Scalability)**: 當需要新增一個也依賴這份資料的圖表元件時，你只需讓它也使用 `useTreeViewModel` Hook，無需改動任何現有程式碼。

### 結論

雖然 Redux 等全功能的狀態管理庫提供了時間旅行、中間件等強大工具，但在許多場景下，**Singleton Service + Custom Hook** 的組合拳提供了一種更輕量、更直觀的解決方案。

它不僅解決了元件間狀態同步的難題，更重要的是，它引導我們寫出結構清晰、職責分離、易於維護和測試的優質程式碼。下次當你面對複雜的狀態共享需求，又不想引入龐大依賴時，不妨試試這個優雅的模式。