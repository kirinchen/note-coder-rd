以下是該教學的繁體中文翻譯版本。我已將其中的程式碼註解翻譯為中文，以幫助理解，並保留了原文的格式。

---

## 如何使用 GitHub Actions 將 React TypeScript 專案部署到 GitHub Pages

**標籤:** `devops`, `React`, `TypeScript`, `git`, `ci/cd`

---

### 如何使用 GitHub Actions 將 React TypeScript 專案部署到 GitHub Pages

#### 簡介 (Introduction)

手動將 React TypeScript 專案部署到 GitHub Pages 可能相當繁瑣。透過使用 GitHub Actions，我們可以自動化此過程，確保你的線上網站始終反映 `main` 分支中的最新變更。

**注意：** 本教學使用現代的 `GITHUB_TOKEN` 方法，這比手動管理個人存取權杖 (Personal Access Tokens) 更加安全且方便。

#### 事前準備 (Prerequisites)

1. **GitHub 帳號**：一個 GitHub 帳號，以及包含你的 React TypeScript 專案的儲存庫 (Repository)。
    
2. **Node.js 和 npm**：確保已安裝 Node.js 和 npm。
    
3. **專案類型**：本指南支援 Vite（推薦）和 Create React App。
    

#### 詳細步驟指南 (Step-by-Step Guide)

**步驟 1：準備你的 React 專案**

你需要告訴你的路由器 (Router) 和打包工具 (Bundler) 應用程式是託管在哪個路徑下。

1. 更新 package.json：
    
    新增 homepage 欄位。這能確保靜態資源 (Assets) 能相對於你的儲存庫 URL 進行正確連結。
    
    JSON
    
    ```
    // 將 'username' 和 'repo-name' 替換為你的實際資訊
    "homepage": "https://username.github.io/repo-name",
    ```
    
2. 對於 Vite 使用者 (重要)：
    
    如果你使用的是 Vite，你必須在 vite.config.ts 中設定 base，使其與你的儲存庫名稱相符：
    
    TypeScript
    
    ```
    // vite.config.ts
    export default defineConfig({
      base: '/your-repo-name/', // 例如：'/my-react-app/'
      plugins: [react()],
    })
    ```
    

**步驟 2：設定 Workflow**

我們不再需要手動產生 Secret 金鑰。我們將使用 GitHub 內建的 `GITHUB_TOKEN`。

1. **建立 Workflow 檔案**：
    
    - 在你的儲存庫中，前往 **Actions** 頁籤。
        
    - 點擊 **New workflow** -> **set up a workflow yourself**。
        
    - 將檔案命名為 `.github/workflows/deploy.yml`。
        
2. 貼上設定內容：
    
    複製以下現代化的 YAML 設定。此腳本使用社群標準的 peaceiris/actions-gh-pages 並設定了適當的寫入權限。
    
    YAML
    
    ```
    name: Deploy to GitHub Pages
    
    on:
      push:
        branches:
          - main
    
    # 授權 GITHUB_TOKEN 寫入儲存庫的權限
    permissions:
      contents: write
    
    jobs:
      deploy:
        runs-on: ubuntu-latest
    
        steps:
          - name: Checkout
            uses: actions/checkout@v4
    
          - name: Setup Node
            uses: actions/setup-node@v4
            with:
              node-version: '20' # 使用現代 LTS 版本
              cache: 'npm'
    
          - name: Install dependencies
            run: npm ci
    
          - name: Build project
            run: npm run build
    
          - name: Deploy
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              # 如果使用 Vite，請用 './dist'。如果使用 CRA，請用 './build'
              publish_dir: ./dist 
    ```
    

**步驟 3：啟用 GitHub Pages**

當 Action 首次成功執行後，會自動建立一個 `gh-pages` 分支。現在你需要告訴 GitHub 從該分支提供網站服務。

1. 前往儲存庫的 **Settings** (設定)。
    
2. 點擊左側側邊欄的 **Pages**。
    
3. 在 **Build and deployment** > **Source** 下方，確保選取 "Deploy from a branch"。
    
4. 在 **Branch** 下方，選擇 `gh-pages` 並確保資料夾設定為 `/(root)`。
    
5. 點擊 **Save** (儲存)。
    

**結論 (Conclusion)**

你的自動化流程 (Pipeline) 現在已經完成！

1. 將程式碼推送到 `main` 分支。
    
2. GitHub Actions 將自動安裝依賴項目、建置你的 React App，並在無需手動 API 金鑰的情況下安全地部署。
    
3. 你的變更將在幾分鐘內上線。
    

你可以在 **Actions** 頁籤中監控部署狀態：

---

**Would you like me to create a diagram explaining the flow of how the GitHub Action interacts with the `gh-pages` branch?**