# Rust 開發環境完整教學

## Windows 11 + WSL2 + Docker + VSCode

> 適用情境：在 Windows 上建立原生 Linux Rust 開發環境，搭配 VSCode 完整 debug 支援，並部署至 AWS Lambda。

---

## 目錄

1. [架構概覽](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#%E6%9E%B6%E6%A7%8B%E6%A6%82%E8%A6%BD)
2. [前置需求](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#%E5%89%8D%E7%BD%AE%E9%9C%80%E6%B1%82)
3. [WSL2 安裝與設定](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#wsl2-%E5%AE%89%E8%A3%9D%E8%88%87%E8%A8%AD%E5%AE%9A)
4. [Docker Desktop 整合](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#docker-desktop-%E6%95%B4%E5%90%88)
5. [VSCode 設定](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#vscode-%E8%A8%AD%E5%AE%9A)
6. [Rust 開發環境](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#rust-%E9%96%8B%E7%99%BC%E7%92%B0%E5%A2%83)
7. [AWS 工具安裝](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#aws-%E5%B7%A5%E5%85%B7%E5%AE%89%E8%A3%9D)
8. [Claude Code CLI](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#claude-code-cli)
9. [第一個 Rust Lambda 專案](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#%E7%AC%AC%E4%B8%80%E5%80%8B-rust-lambda-%E5%B0%88%E6%A1%88)
10. [GitHub Actions CD Pipeline](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#github-actions-cd-pipeline)
11. [安全性最佳實踐](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#%E5%AE%89%E5%85%A8%E6%80%A7%E6%9C%80%E4%BD%B3%E5%AF%A6%E8%B8%90)
12. [常見問題](https://claude.ai/chat/e52ce53f-db73-45e4-97e8-009e3d628752#%E5%B8%B8%E8%A6%8B%E5%95%8F%E9%A1%8C)

---

## 架構概覽

```
Windows 11
├── VSCode (Windows App)
│   └── WSL Extension → 連接 WSL2 環境
├── Docker Desktop
│   └── 底層使用 WSL2 運行容器
└── WSL2 (Ubuntu 24.04)
    ├── Rust + cargo + cargo-lambda
    ├── AWS CLI + SAM CLI
    ├── Claude Code CLI
    ├── Node.js (via nvm)
    └── Git + gh CLI
```

**為什麼選 WSL2？**

- Lambda 跑在 Linux，開發環境與目標環境一致，cross-compile 不需要額外工具
- VSCode WSL Extension 支援完整 breakpoint debug
- 檔案 I/O 速度比 `/mnt/c/` 快 3-5 倍
- 所有 Linux 工具開箱即用

---

## 前置需求

|需求|最低版本|確認指令|
|---|---|---|
|Windows|11 或 10 (Build 19041+)|`winver`|
|PowerShell|5.1+|`$PSVersionTable`|
|Docker Desktop|4.48+|Docker Desktop → About|
|VSCode|最新版|`code --version`|

---

## WSL2 安裝與設定

### Step 1：確認 WSL2 已安裝

在 **PowerShell（系統管理員）** 執行：

```powershell
wsl --version
```

預期輸出：

```
WSL 版本: 2.x.x
核心版本: x.x.x
```

### Step 2：安裝 Ubuntu

```powershell
wsl --install -d Ubuntu
```

安裝完成後系統會提示設定：

- **Username**：輸入你的 Linux 使用者名稱（小寫英文）
- **Password**：設定密碼（輸入時不會顯示，這是正常的）

### Step 3：確認 Ubuntu 版本

```bash
# 在 WSL2 Ubuntu 裡執行
lsb_release -a
```

### Step 4：更新系統套件

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 5：安裝基本開發工具

```bash
sudo apt install -y \
  build-essential \
  curl \
  git \
  unzip \
  pkg-config \
  libssl-dev
```

### Step 6：設定 Git

```bash
git config --global user.name "你的名字"
git config --global user.email "你的信箱@example.com"
git config --global init.defaultBranch main
```

---

## Docker Desktop 整合

### 確認 Docker 在 WSL2 中可用

```bash
# 在 WSL2 Ubuntu 裡執行
docker --version
docker run hello-world
```

如果出現 permission denied：

```bash
sudo usermod -aG docker $USER
# 重新開啟 WSL2 terminal
```

### Docker Desktop 設定

開啟 Docker Desktop → Settings → Resources → WSL Integration：

- 勾選 **Enable integration with my default WSL distro**
- 勾選 **Ubuntu**

---

## VSCode 設定

### Step 1：安裝必要 Extension（在 Windows VSCode 裡）

| Extension        | ID                            | 用途            |
| ---------------- | ----------------------------- | ------------- |
| WSL              | `ms-vscode-remote.remote-wsl` | 連接 WSL2       |
| rust-analyzer    | `rust-lang.rust-analyzer`     | Rust 語言支援     |
| CodeLLDB         | `vadimcne.codelldb`           | Rust Debug    |
| Even Better TOML | `tamasfe.even-better-toml`    | Cargo.toml 支援 |
| GitLens          | `eamodio.gitlens`             | Git 增強        |

### 什麼是「WSL2 終端機」？如何開啟？

**WSL2 終端機**指的是在 **Linux（Ubuntu）環境** 裡跑 shell（例如 Bash）的視窗；看到的提示字元像 `user@hostname:~$`，指令與路徑都是 **Linux 語意**（`~`、`/`、`sudo`），不是 PowerShell。

常用開啟方式（擇一即可）：

| 方式 | 操作 |
| --- | --- |
| Ubuntu 應用程式 | **開始** 選單搜「Ubuntu」→ 開啟 |
| Windows Terminal | 開啟 Windows Terminal → 分頁下拉選 **Ubuntu**（若沒有，先到設定裡新增 WSL 設定檔） |
| 從 PowerShell 進入 | 在 PowerShell 執行 `wsl` 或 `wsl -d Ubuntu` |
| 已在 VSCode 用 Remote-WSL | 連上 WSL 後，**終端機 → 新增終端機** 預設就是該 distro 的 shell |

第一次若跳出 Linux 帳密登入，代表你已進入 WSL2，不是 Windows 的 `C:\`。

### Step 2：從 WSL2 開啟 VSCode（需先有專案資料夾）

`code .` 的意義是：**用目前目錄當成工作區** 在 Windows 開啟 VSCode，並透過 Remote-WSL 掛到 WSL2。因此目錄必須已存在；本文件的閱讀順序裡，**完整的 Lambda 專案要到後面 [第一個 Rust Lambda 專案](#第一個-rust-lambda-專案) 才用 `gh repo create` 等方式建立**。

- **建議流程**：先照順序完成 [Rust 開發環境](#rust-開發環境)，再完成「第一個 Rust Lambda 專案」的建庫步驟；在 WSL2 終端機裡 **`cd` 進專案根目錄**（`gh repo create ... --clone` 預設會在目前目錄下出現 `hello-lambda-rust`，故常見路徑為 `~/hello-lambda-rust`；若你改在 `~/projects` 下執行建庫，則為 `~/projects/hello-lambda-rust`），然後再執行下面的 `code .`。
- **若只想先確認 VSCode ↔ WSL 能連線**：可在 WSL2 終端機建立空資料夾測試，不一定要先有 Rust 專案：

```bash
mkdir -p ~/projects/vscode-wsl-smoke-test
cd ~/projects/vscode-wsl-smoke-test
code .
```

附註：若你在 Windows 已安裝 VSCode 並勾選「將 `code` 加入 PATH」，在 WSL2 裡通常也能執行 `code`；若指令找不到，請在 Windows 端重新安裝 VSCode 並啟用 **Shell Command: Install 'code' command in PATH**。

```bash
# 實際開發時：在 WSL2 終端機進入「已存在的」專案根目錄後執行
# 例：Lambda 專案建在本機家目錄時
cd ~/hello-lambda-rust

# 在 Windows 開啟 VSCode，但連接到 WSL2（Remote-WSL）
code .
```

VSCode 左下角會顯示：

```
><WSL: Ubuntu
```

### Step 3：設定 Debug（launch.json）

在專案根目錄建立 `.vscode/launch.json`：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug Lambda Handler",
      "cargo": {
        "args": ["build", "--bin", "bootstrap", "--package", "hello-lambda-rust"],
        "filter": {
          "name": "bootstrap",
          "kind": "bin"
        }
      },
      "args": [],
      "cwd": "${workspaceFolder}"
    },
    {
      "type": "lldb",
      "request": "launch",
      "name": "Debug Unit Tests",
      "cargo": {
        "args": ["test", "--no-run", "--lib"],
        "filter": {
          "name": "hello-lambda-rust",
          "kind": "lib"
        }
      },
      "args": [],
      "cwd": "${workspaceFolder}"
    }
  ]
}
```

### Step 4：設定 rust-analyzer（settings.json）

在 `.vscode/settings.json`：

```json
{
  "rust-analyzer.cargo.buildScripts.enable": true,
  "rust-analyzer.procMacro.enable": true,
  "editor.formatOnSave": true,
  "[rust]": {
    "editor.defaultFormatter": "rust-lang.rust-analyzer"
  }
}
```

---

## Rust 開發環境

### Step 1：安裝 Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

選擇 `1) Proceed with standard installation`。

```bash
# 套用環境變數
source ~/.bashrc

# 確認安裝
rustc --version
cargo --version
```

### Step 2：安裝 Lambda 編譯 target

```bash
rustup target add x86_64-unknown-linux-musl
```

### Step 3：安裝 cargo-lambda

```bash
pip3 install cargo-lambda --break-system-packages

# 確認安裝
cargo lambda --version
```

### Step 4：安裝常用 cargo 工具

```bash
# 程式碼格式化（通常已內建）
rustup component add rustfmt

# 程式碼 lint
rustup component add clippy

# 安全性稽核
cargo install cargo-audit
```

---

## AWS 工具安裝

### Step 1：安裝 AWS CLI v2

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# 清理
rm -rf awscliv2.zip aws/

# 確認
aws --version
```

### Step 2：設定 AWS Credentials

```bash
aws configure
# AWS Access Key ID: 輸入你的 Access Key
# AWS Secret Access Key: 輸入你的 Secret Key
# Default region name: ap-northeast-1
# Default output format: json
```

### Step 3：安裝 AWS SAM CLI

```bash
pip3 install aws-sam-cli --break-system-packages

# 確認
sam --version
```

---

## Claude Code CLI

### Step 1：安裝 Node.js（透過 nvm）

```bash
# 安裝 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc

# 安裝 LTS 版本
nvm install --lts

# 確認
node --version   # v22.x.x
npm --version
```

### Step 2：安裝 Claude Code

```bash
npm install -g @anthropic-ai/claude-code

# 確認
claude --version
```

### Step 3：登入（只需一次）

```bash
claude login
# 會開啟瀏覽器，用 Anthropic 帳號登入
```

### Step 4：設定 MCP Servers

MCP 讓 Claude Code 能直接操作外部工具，設定後所有 project 都能使用。

**GitHub MCP：**

> ⚠️ 以下指令直接在 terminal 執行，不要把真實 token 貼進 Claude Code 對話。

```bash
# 先去 github.com → Settings → Developer settings → Personal access tokens
# 產生新 token，勾選 repo、workflow 權限

claude mcp add-json github '{
  "type": "http",
  "url": "https://api.githubcopilot.com/mcp",
  "headers": {"Authorization": "Bearer 你的GITHUB_PAT"}
}'
```

**確認 MCP 已設定：**

```bash
claude mcp list
```

### MCP 層級說明

|層級|指令|存放位置|範圍|
|---|---|---|---|
|user（預設）|`--scope user`|`~/.claude.json`|所有 project|
|local|`--scope local`|`.claude/settings.json`|此 project，不 commit|
|project|`--scope project`|`.mcp.json`|此 project，會 commit|

---

## 第一個 Rust Lambda 專案

### Step 1：建立 GitHub Repo

```bash
# 安裝 gh CLI
sudo apt install gh -y

# 登入 GitHub
gh auth login

# 建立 public repo 並 clone
gh repo create hello-lambda-rust --public --clone
cd hello-lambda-rust
```

### Step 2：進入 Claude Code

```bash
claude
```

```
> /init
```

然後輸入需求：

```
我想建立一個 Rust Hello World AWS Lambda project
環境：WSL2 Ubuntu
工具：cargo-lambda + AWS SAM + CloudFormation
CD：GitHub Actions + OIDC（不使用長期 Access Key）
Region：ap-northeast-1

請幫我建立：
1. src/main.rs（Lambda handler）
2. Cargo.toml
3. template.yaml（SAM template）
4. Makefile（本地部署指令）
5. .github/workflows/deploy.yml（GitHub Actions）
6. .gitignore
```

### Step 3：專案結構

```
hello-lambda-rust/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── .vscode/
│   ├── launch.json
│   └── settings.json
├── src/
│   └── main.rs
├── Cargo.toml
├── template.yaml
├── Makefile
├── CLAUDE.md
└── .gitignore
```

### Step 4：核心程式碼

**`src/main.rs`：**

```rust
use lambda_runtime::{run, service_fn, Error, LambdaEvent};
use serde::{Deserialize, Serialize};

#[derive(Deserialize)]
struct Request {}

#[derive(Serialize)]
struct Response {
    message: String,
}

async fn handler(event: LambdaEvent<Request>) -> Result<Response, Error> {
    let (_, _context) = event.into_parts();
    Ok(Response {
        message: "Hello, World from Rust Lambda!".to_string(),
    })
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    run(service_fn(handler)).await
}
```

**`Cargo.toml`：**

```toml
[package]
name = "hello-lambda-rust"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "bootstrap"
path = "src/main.rs"

[dependencies]
lambda_runtime = "0.13"
serde = { version = "1", features = ["derive"] }
tokio = { version = "1", features = ["macros"] }
```

**`template.yaml`：**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Timeout: 10
    MemorySize: 128

Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: hello-lambda-rust
      CodeUri: target/lambda/bootstrap/
      Handler: bootstrap
      Runtime: provided.al2023
      Architectures: [x86_64]
```

**`Makefile`：**

```makefile
STACK_NAME  = hello-lambda-rust
REGION      = ap-northeast-1
S3_BUCKET   = your-deployment-bucket

.PHONY: build deploy destroy invoke

build:
	cargo lambda build --release

invoke:
	cargo lambda invoke hello-lambda-rust --data-ascii '{}'

deploy: build
	sam package \
		--template-file template.yaml \
		--s3-bucket $(S3_BUCKET) \
		--output-template-file packaged.yaml \
		--region $(REGION)
	sam deploy \
		--template-file packaged.yaml \
		--stack-name $(STACK_NAME) \
		--capabilities CAPABILITY_IAM \
		--region $(REGION) \
		--no-confirm-changeset

destroy:
	aws cloudformation delete-stack \
		--stack-name $(STACK_NAME) \
		--region $(REGION)
```

### Step 5：本地測試

開兩個 WSL2 terminal：

```bash
# Terminal 1：啟動本地 Lambda 模擬器
cargo lambda watch

# Terminal 2：發送測試請求
cargo lambda invoke hello-lambda-rust --data-ascii '{}'
```

### Step 6：第一次部署

```bash
# 建立 S3 bucket（只需一次）
aws s3 mb s3://my-lambda-deploy-$(whoami) --region ap-northeast-1

# 部署
make deploy
```

---

## GitHub Actions CD Pipeline

### Step 1：在 AWS 建立 OIDC Provider（只需一次）

```bash
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com \
  --thumbprint-list 6938fd4d98bab03faadb97b34396831e3780aea1
```

### Step 2：建立 IAM Role

建立 `trust-policy.json`：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_USERNAME/hello-lambda-rust:*"
        }
      }
    }
  ]
}
```

```bash
# 建立 Role
aws iam create-role \
  --role-name github-actions-lambda-role \
  --assume-role-policy-document file://trust-policy.json

# 附加權限（依需求調整）
aws iam attach-role-policy \
  --role-name github-actions-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/AWSCloudFormationFullAccess

aws iam attach-role-policy \
  --role-name github-actions-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/AWSLambda_FullAccess

aws iam attach-role-policy \
  --role-name github-actions-lambda-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### Step 3：設定 GitHub Repository Variables

在 GitHub repo → Settings → Secrets and variables → Actions：

- **Variables（公開）**：
    - `S3_BUCKET` = 你的 S3 bucket 名稱
    - `AWS_REGION` = `ap-northeast-1`

### Step 4：deploy.yml

```yaml
name: Deploy to AWS Lambda

on:
  push:
    branches: [main]

permissions:
  id-token: write   # OIDC 必須
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::YOUR_ACCOUNT_ID:role/github-actions-lambda-role
          aws-region: ${{ vars.AWS_REGION }}

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable
        with:
          targets: x86_64-unknown-linux-musl

      - name: Cache cargo
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target/
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}

      - name: Install cargo-lambda
        run: pip install cargo-lambda

      - name: Build
        run: cargo lambda build --release

      - name: Deploy via SAM
        run: |
          sam package \
            --template-file template.yaml \
            --s3-bucket ${{ vars.S3_BUCKET }} \
            --output-template-file packaged.yaml \
            --region ${{ vars.AWS_REGION }}
          sam deploy \
            --template-file packaged.yaml \
            --stack-name hello-lambda-rust \
            --capabilities CAPABILITY_IAM \
            --region ${{ vars.AWS_REGION }} \
            --no-confirm-changeset
```

---

## 安全性最佳實踐

### AWS 金鑰管理

|方法|安全等級|說明|
|---|---|---|
|寫死在程式碼|❌ 絕對禁止|會直接暴露|
|放進 GitHub Secrets|✅ 可接受|適合快速開始|
|OIDC + IAM Role|✅✅ 最佳|無長期金鑰，推薦正式使用|

### Secret 處理原則

```bash
# ✅ 安全：直接在 terminal 執行
claude mcp add-json github '{"headers": {"Authorization": "Bearer ghp_xxxx"}}'

# ❌ 危險：貼進 Claude Code 或任何 AI 對話
你：幫我設定 MCP，token 是 ghp_xxxx...
```

### .gitignore 必備內容

```gitignore
# Rust build 產出
/target/
packaged.yaml

# AWS credentials（萬一有 .env 檔）
.env
.env.local
.env.*.local

# Claude Code 本地設定
.claude/settings.local.json

# IDE
.vscode/settings.json   # 如果有個人設定不想 commit
```

---

## 常見問題

### WSL2 找不到 Ubuntu

```powershell
# PowerShell
wsl --list --verbose
# 如果沒有 Ubuntu，重新安裝
wsl --install -d Ubuntu
```

### cargo lambda build 失敗（找不到 linker）

```bash
sudo apt install -y musl-tools
rustup target add x86_64-unknown-linux-musl
```

### VSCode 沒有顯示 WSL 連接

```bash
# 確認在 WSL2 terminal 裡執行
code .
# 不是在 Windows PowerShell 執行
```

### claude login 後瀏覽器沒有開啟

```bash
# 手動複製 URL 在 Windows 瀏覽器開啟
# 或設定 BROWSER 環境變數
export BROWSER=wslview
claude login
```

### Docker 在 WSL2 裡無法執行

```bash
# 加入 docker group
sudo usermod -aG docker $USER
# 重新啟動 WSL2
wsl --shutdown
# 重新開啟 Ubuntu
```

### SAM deploy 失敗：S3 bucket 不存在

```bash
aws s3 mb s3://你的-bucket-名稱 --region ap-northeast-1
```

---

## 環境確認清單

全部裝好後，執行這個確認所有工具版本：

```bash
echo "=== Rust ===" && rustc --version && cargo --version
echo "=== cargo-lambda ===" && cargo lambda --version
echo "=== Node.js ===" && node --version
echo "=== Claude Code ===" && claude --version
echo "=== AWS CLI ===" && aws --version
echo "=== SAM CLI ===" && sam --version
echo "=== Git ===" && git --version
echo "=== Docker ===" && docker --version
echo "=== gh CLI ===" && gh --version
```

全部有輸出版本號，環境就設定完成了 🎉

---

_文件版本：2026-04  
適用環境：Windows 11 + WSL2 Ubuntu 24.04 + Docker Desktop 4.48+_