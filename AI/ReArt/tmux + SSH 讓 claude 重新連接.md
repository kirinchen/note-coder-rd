# tmux + SSH 讓 claude 重新連接

> 透過 SSH 連線到 Linux Mint，搭配 tmux 讓 Claude Code 回測任務在斷線後繼續執行。

---

## 架構說明

```
Linux Mint（遠端主機）          你的電腦
┌─────────────────────┐         ┌──────────────┐
│  tmux session       │◄──SSH───│  terminal    │
│  ┌───────────────┐  │         └──────────────┘
│  │ claude        │  │
│  │ (回測優化任務) │  │    SSH 斷線？
│  └───────────────┘  │    → tmux 繼續運行
└─────────────────────┘    → 重新 SSH + attach 即可
```

**核心觀念：** SSH 斷線只是斷掉「你和主機之間的連線」，tmux 跑在主機本機記憶體裡，完全不受影響。Claude Code 的回測任務會繼續執行直到完成。

---

## Step 1 — 確認 SSH Server 已安裝並啟動

> 在 Linux Mint 上執行

```bash
# 安裝 openssh-server（通常 Linux Mint 已有）
sudo apt update
sudo apt install -y openssh-server

# 啟動並設為開機自動啟動
sudo systemctl enable ssh
sudo systemctl start ssh

# 確認狀態
sudo systemctl status ssh
```

看到 **`Active: active (running)`** 就代表 SSH server 正常運作。

記下 Linux Mint 的 IP：

```bash
ip addr show
```

---

## Step 2 — 安裝 tmux

> 在 Linux Mint 上執行

tmux 是 terminal multiplexer，讓你的 session 在 SSH 斷線後繼續存活，是長時間回測任務的必備工具。

```bash
sudo apt install -y tmux

# 確認版本
tmux -V
# 應看到 tmux 3.x
```

---

## Step 3 — 從你的電腦 SSH 連入 Linux Mint

> 在你的電腦上執行

```bash
# 基本 SSH 連線
ssh your_username@192.168.x.x

# 如果要指定 port（預設 22）
ssh -p 22 your_username@192.168.x.x

# 建議設定 SSH key 免密碼登入（更安全方便）
ssh-keygen -t ed25519
ssh-copy-id your_username@192.168.x.x
```

---

## Step 4 — 建立 tmux session 並啟動 Claude Code

> SSH 進去後執行

```bash
# 建立新的 tmux session，命名為 quant（或任何你喜歡的名字）
tmux new-session -s quant

# 進入 session 後，cd 到你的量化系統目錄
cd ~/your-quant-system

# 啟動 Claude Code
claude
```

> 現在 Claude Code 正在運行，你可以給它指令去跑回測、優化特徵參數組合。這個 session 即使 SSH 斷線，也會繼續在背景跑。

---

## Step 5 — SSH 斷線後重新接管 session

不管是網路中斷、筆電睡眠、或主動關掉 terminal，只要 Linux Mint 主機沒有關機，tmux session 就還活著。

```bash
# 重新 SSH 進去
ssh your_username@192.168.x.x

# 查看目前有哪些 tmux sessions 還在跑
tmux ls
# 會看到類似：quant: 1 windows (created ...)

# 重新接管（attach）你的 session
tmux attach -t quant

# 就會直接看到 Claude Code 繼續跑的畫面
```

> ⚠️ 只要 Linux Mint 本機不關機、不重開機，tmux session 就永遠存在，不受 SSH 連線影響。

---

## 常用 tmux 快捷鍵

**前綴鍵：`Ctrl + b`，放開後再按指令鍵**

|快捷鍵|功能|
|---|---|
|`Ctrl+b` → `d`|Detach（主動離開 session，session 繼續跑）|
|`Ctrl+b` → `$`|重新命名 session|
|`Ctrl+b` → `c`|在同一 session 新開一個 window|
|`Ctrl+b` → `n` / `p`|切換到下一個 / 上一個 window|
|`Ctrl+b` → `%`|垂直分割 pane（左右兩個終端）|
|`Ctrl+b` → `"`|水平分割 pane（上下兩個終端）|
|`Ctrl+b` → `[`|進入 scroll 模式（可往上翻 log，`q` 退出）|
|`tmux kill-session -t quant`|手動結束指定的 session|

---

## 進階：同時跑多組回測（多個 Claude 實例）

如果你有多組特徵組合要並行測試，可以開多個 tmux session，各自跑一個 Claude Code 實例。

```bash
# 開多個 session，分別做不同任務
# -d 是 detached 模式，直接背景啟動
tmux new-session -d -s backtest-1
tmux new-session -d -s backtest-2
tmux new-session -d -s optimize

# 查看所有 session
tmux ls

# 切換到某個 session
tmux attach -t backtest-1

# 在不 attach 的情況下，直接對某 session 送指令
tmux send-keys -t backtest-2 'claude' Enter
```

> 多個 Claude Code 實例並行跑不同特徵組合，結果各自輸出到不同的 log 或資料夾，是量化回測加速的常見做法。

---

## 完整流程速查

```bash
## 第一次設定（只需做一次）
sudo apt install -y openssh-server tmux
sudo systemctl enable --now ssh

## 每次開始回測任務
ssh user@192.168.x.x
tmux new-session -s quant
cd ~/your-quant-system
claude

## SSH 斷線後重新接管
ssh user@192.168.x.x
tmux attach -t quant
```

---

## 補充說明

- **tmux session 跨重開機存活**：預設 tmux session 不能跨主機重開機存活。如需此功能，可安裝 `tmux-resurrect` 外掛。量化任務通常在數小時至一天內完成，一般不需要。
- **遠端訪問**：如果你不在同一個區網，可以搭配 Tailscale 或設定 port forwarding，從外網也能 SSH 進來接管任務。
- **Claude Code Remote Control**：如果你有 Claude Pro/Max 訂閱，也可以搭配 `claude remote-control` 指令，透過手機或瀏覽器監看 Claude Code 的執行狀況，和 tmux 並不衝突。