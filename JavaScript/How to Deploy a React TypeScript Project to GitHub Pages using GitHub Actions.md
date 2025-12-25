---
title: How to Deploy a React TypeScript Project to GitHub Pages using GitHub Actions
tags: [devops, React, TypeScript, git, ci/cd]

---

### How to Deploy a React TypeScript Project to GitHub Pages using GitHub Actions

#### Introduction

Deploying a React TypeScript project to GitHub Pages manually can be tedious. By using GitHub Actions, we can automate this process to ensure your live site always reflects the latest changes in your main branch. 

**Note:** This tutorial uses the modern `GITHUB_TOKEN` method, which is more secure than managing personal access tokens manually.

#### Prerequisites

1.  **GitHub Account**: A GitHub account and a repository containing your React TypeScript project.
2.  **Node.js and npm**: Ensure Node.js and npm are installed.
3.  **Project Type**: This guide supports both Vite (recommended) and Create React App.

#### Step-by-Step Guide

**Step 1: Prepare Your React Project**

You need to tell your router and bundler where the app is hosted.

1.  **Update `package.json`**:
    Add the `homepage` field. This ensures assets are linked correctly relative to your repository URL.

    ```json
    // Replace 'username' and 'repo-name' with your actual details
    "homepage": "[https://username.github.io/repo-name](https://username.github.io/repo-name)",
    ```

2.  **For Vite Users (Important)**:
    If you are using Vite, you must also set the `base` in `vite.config.ts` to match your repository name:

    ```typescript
    // vite.config.ts
    export default defineConfig({
      base: '/your-repo-name/', // e.g. '/my-react-app/'
      plugins: [react()],
    })
    ```

**Step 2: Configure the Workflow**

We no longer need to generate manual secrets. We will use the built-in `GITHUB_TOKEN`.

1.  **Create a Workflow File**:
    - In your repository, navigate to the **Actions** tab.
    - Click **New workflow** -> **set up a workflow yourself**.
    - Name the file `.github/workflows/deploy.yml`.

2.  **Paste the Configuration**:
    Copy the following modern YAML configuration. This script uses the community-standard `peaceiris/actions-gh-pages` and sets appropriate permissions.

    ```yaml
    name: Deploy to GitHub Pages

    on:
      push:
        branches:
          - main

    # Grant GITHUB_TOKEN the permission to write to the repository
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
              node-version: '20' # Use a modern LTS version
              cache: 'npm'

          - name: Install dependencies
            run: npm ci

          - name: Build project
            run: npm run build

          - name: Deploy
            uses: peaceiris/actions-gh-pages@v4
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              # If using Vite, use './dist'. If using CRA, use './build'
              publish_dir: ./dist 
    ```

**Step 3: Enable GitHub Pages**

Once the action runs successfully for the first time, a `gh-pages` branch will be created automatically. Now you need to tell GitHub to serve your site from there.

1.  Go to your repository **Settings**.
2.  Click on **Pages** in the left sidebar.
3.  Under **Build and deployment** > **Source**, ensure "Deploy from a branch" is selected.
4.  Under **Branch**, select `gh-pages` and ensuring the folder is `/(root)`.
5.  Click **Save**.

**Conclusion**

Your automation pipeline is now complete! 

1.  Push code to the `main` branch.
2.  GitHub Actions will automatically install dependencies, build your React app, and deploy it securely without requiring manual API keys.
3.  Your changes will be live in a few minutes.

You can monitor the deployment status in the **Actions** tab:

![Action Status](https://hackmd.io/_uploads/HkYORjz-0.png)