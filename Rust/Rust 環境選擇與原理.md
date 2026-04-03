# Rust 環境選擇與原理

> 在 Windows 上常同時遇到：**本機桌面程式（例如 Tauri）**與**部署到 Linux（例如 AWS Lambda）**。這篇說明 `rustup`、編譯目標（target）與「在哪裝 Rust」之間的關係，方便你決定用 **Windows 原生**、**WSL2**，還是兩者並存。

相關筆記：

- [Rust 開發環境完整教學](./Rust%20開發環境完整教學.md)（WSL2 + Lambda 為主線）
- [打造 Tauri v2 專案基礎 Hello World](./Tauri/打造%20Tauri%20v2%20專案基礎%20Hello%20World.md)（Windows + MSVC 為主線）

---

## 1. 先釐清三個層次

| 層次 | 是什麼 | 你實際遇到的名字 |
|------|--------|------------------|
| **Rust 工具鏈（toolchain）** | 某個通道的 `rustc` + `cargo` + 標準函式庫，綁定 **host**（你正在執行編譯器的那台機器的 OS/arch） | `stable-x86_64-pc-windows-msvc`、`stable-x86_64-unknown-linux-gnu` |
| **編譯目標（target）** | **產出來的二進位**要跑在哪種 OS/ABI | `x86_64-pc-windows-msvc`、`x86_64-unknown-linux-gnu`、`aarch64-unknown-linux-gnu` 等 |
| **開發環境所在** | 你在哪裡開終端機、掛哪個檔案系統、裝哪些系統套件 | Windows PowerShell、WSL2 Ubuntu、Docker、CI 上的 Linux |

**原理重點：** `cargo build` 預設會建 **「host = target」** 的產物。你要產出「別種系統能跑的 exe 或 ELF」，就變成 **cross-compile**（或改名為同一台機器上的「非預設 target」），通常要額外的 **linker**、**C 函式庫**、有時還有 **SDK**。

---

## 2. Windows 本機：MSVC 與 GNU 兩條線

在 **Windows** 上，`rustup` 最常見的 default host 是：

- **`x86_64-pc-windows-msvc`**：用微軟 **MSVC** 工具鏈（`link.exe`、Windows SDK）。**Tauri、多數 Windows 商業軟體生態**預期你走這條。
- **`x86_64-pc-windows-gnu`**：用 **MinGW-w64**。較少用於 Tauri；某些純 CLI 或依賴情境會選，但與「正版微軟連結器 + WebView2」路線不同。

**原理：** Rust 產生目標平台的機器碼之後，還要 **連結（link）** 成最終 executable。在 Windows 上，MSVC 目標依賴微軟提供的連結器與 CRT；GNU 目標則依賴另一套 mingw 工具。混用或終端機 PATH 指到錯的 `link.exe`（例如 Git Bash 的同名工具）就會出現各種詭異錯誤——这也是 Tauri 筆記強調 **PowerShell / CMD** 的原因。

---

## 3. WSL2：Linux host，但人還在 Windows 桌上

WSL2 裡的典型 host 是 **`x86_64-unknown-linux-gnu`**（或 ARM 機器則是對應的 `aarch64-unknown-linux-gnu`）。

**原理：** WSL2 是 **貨真價實的 Linux 核心 + Linux 使用者空間**（發行版如 Ubuntu），不是「模擬跑 Rust」，而是 **另一套 OS 上的另一份 Rust**。與「在實體機或 VM 裝 Ubuntu」對編譯行為而言，同一概念。

實務上常聽到的差異：

- **檔案路徑**：專案放在 WSL 的 `~/project` 比放在 `/mnt/c/...` 底下 **I/O 通常快很多**，大型專案編譯差異明顯。
- **與部署環境一致**：目標若是 **Linux 伺服器 / 容器 / Lambda**，在 WSL2 用與正式環境相近的 glibc / 工具，**除錯「只在我機器過、上線爆」** 的機率較低。
- **GUI 桌面程式**：在 WSL2 跑 **Linux 版** 桌面程式可行（例如透過 WSLg），但 **Windows 版 .exe** 仍是以 Windows 工具鏈生產為主；從純 WSL 交叉產出 **Windows MSVC exe** 需要額外設定，不適合當入門預設路徑。

---

## 4. 依「產品跑在哪裡」選環境（決策表）

| 你的產出 | 建議主力環境 | 簡短原理 |
|----------|--------------|----------|
| **Windows 桌面（Tauri、WinUI 等）** | Windows + **MSVC** toolchain | 連結器、WebView2、Windows API 與除錯流程都以 Windows 原生為準。 |
| **Linux 伺服器 / Lambda / 容器內二進位** | **WSL2** 或 Linux VM / CI | Host 與 target 皆 Linux，`cargo`、動態連結、腳本、檔案權限與正式環境一致。 |
| **幾乎只靠 CI 建 Linux，本機只寫碼** | Windows -only 也行 | 本機不建 Linux artifact，負擔在 CI；本機仍要能跑測試的話通常還是 WSL 或 Docker 較順。 |
| **堅持只有一份 Rust（全在 Windows）又要 Linux 二進位** | 須處理 **cross-compile**（target + linker + sysroot） | 可行但學習曲線陡，不如 WSL2「直接當 Linux 開發」直觀。 |

---

## 5. 兩套 Rust 並存算不算浪費？

**不算。** Windows 上一份 `rustup`、WSL2 裡再一份 `rustup`，是 **兩個 OS 環境各自一份工具**，就像你在筆電裝 Windows、伺服器裝 Linux 各裝一次一樣。

- **磁碟與維護**：確實多一份 toolchain；但以現今容量與 `rustup` 管理，通常可接受。
- **心智模型**：**Tauri → Windows 路徑**；**Lambda / Linux 服務 → WSL2 路徑**。路徑分開，反而少踩連結器與 target 混淆。

---

## 6. 常用 self-check 指令（釐清你現在是哪一套）

在 **PowerShell（Windows）**：

```powershell
rustc -vV
```

看 *host* 是否為 `x86_64-pc-windows-msvc`（或你安裝的 windows target）。

在 **WSL2 Ubuntu**：

```bash
rustc -vV
```

*host* 應為 `x86_64-unknown-linux-gnu`（或對應架構）。

檢查已安裝的 target：

```bash
rustup target list --installed
```

---

## 7. 一句話總結

- **環境選擇的本質**是：**編譯器 host**、**產物 target**、**執行時真正的 OS** 三者要對齊；對齊則順，不對齊就要 cross-compile 或換環境。
- **Windows 桌面** → 優先用 **本機 Windows + MSVC**。
- **Linux 上線** → 優先在 **WSL2（或 Linux）** 開發與編譯。
- **兩邊專案都有** → **兩套 Rust 並存**是正常、務實的做法，不必為了「只留一份」而犧牲某一類專案的體驗。

---

## 延伸閱讀（官方概念）

- [Rustup book：Cross-compilation](https://rust-lang.github.io/rustup/cross-compilation.html) — target 與 `rustup target add`。
- [Reference：Platform support](https://doc.rust-lang.org/nightly/rustc/platform-support.html) — 各 target 層級（tier）與支援狀態。
