# tmux 速查表

> 配套文件:[Ambient Coding 技術 Spec](https://claude.ai/chat/ambient-coding-spec.md)  
> 適用版本:tmux 3.0+

---

## 🎯 必須先懂的觀念

### Prefix Key(前綴鍵)

tmux 的所有快捷鍵都是「**Prefix → 鍵**」的兩段式:

```
按住 Ctrl 同時按 B  →  放開兩個鍵  →  再按功能鍵
```

預設 Prefix = `Ctrl+B`(本文件以 `<Prefix>` 表示)。

### 三層架構

```
Session(會議室)
  └── Window(房間)            ← 像瀏覽器分頁
        └── Pane(房間內的桌)   ← 像螢幕分割
```

**新手只要先學會 Session 操作即可**,Window 和 Pane 是進階。

---

## 📦 Part 1:CLI 指令(在 tmux 外面用)

### Session 基本操作

| 指令                            | 功能                                     |
| ----------------------------- | -------------------------------------- |
| `tmux`                        | 建立一個無名 session(不推薦,難管理)                |
| `tmux new -s <name>`          | 建立指定名稱 session,例如 `tmux new -s claude` |
| `tmux ls`                     | 列出所有 session                           |
| `tmux attach`                 | 進入最近使用的 session                        |
| `tmux attach -t <name>`       | 進入指定 session                           |
| `tmux a -t <name>`            | 同上,簡寫                                  |
| `tmux kill-session -t <name>` | 殺掉指定 session                           |
| `tmux kill-server`            | 殺掉所有 session(慎用)                       |

### 進階建立

|指令|功能|
|---|---|
|`tmux new -s <name> -d`|在背景建立 session,不進去|
|`tmux new -s <name> "<command>"`|建立 session 並直接跑指令,例如 `tmux new -s build "npm run dev"`|
|`tmux new -As <name>`|若 session 存在就 attach,不存在就建立(超實用)|

### 強制踢掉其他 client

|指令|功能|
|---|---|
|`tmux attach -t <name> -d`|attach 並踢掉其他已 attach 的 client(避免畫面被拉小)|

---

## ⌨️ Part 2:Attach 後的快捷鍵

> 全部以 `<Prefix>` 開頭,即先按 `Ctrl+B` 放開,再按下面的鍵。

### 🔑 最常用的 5 個(新手只記這些就夠)

|快捷鍵|功能|重要度|
|---|---|---|
|`<Prefix> d`|**D**etach,離開 session(裡面繼續跑)|⭐⭐⭐⭐⭐|
|`<Prefix> s`|列出所有 **S**ession,上下選|⭐⭐⭐⭐⭐|
|`<Prefix> [`|進入 scroll mode,可上下捲看歷史輸出(`q` 離開)|⭐⭐⭐⭐⭐|
|`<Prefix> ?`|顯示**所有快捷鍵**列表(忘了就按這個)|⭐⭐⭐⭐|
|`<Prefix> :`|進入 command mode,可手打 tmux 指令|⭐⭐⭐|

### 🪟 Session 操作

|快捷鍵|功能|
|---|---|
|`<Prefix> d`|Detach 離開(session 繼續跑)|
|`<Prefix> s`|開 session 列表選擇|
|`<Prefix> $`|重新命名當前 session|
|`<Prefix> (`|切換到上一個 session|
|`<Prefix> )`|切換到下一個 session|
|`<Prefix> L`|切回剛剛那個 session(類似 `cd -`)|

### 🪟 Window 操作(像分頁)

|快捷鍵|功能|
|---|---|
|`<Prefix> c`|**C**reate 新 window|
|`<Prefix> ,`|重新命名當前 window|
|`<Prefix> &`|關閉當前 window(會問你 y/n)|
|`<Prefix> n`|**N**ext window|
|`<Prefix> p`|**P**revious window|
|`<Prefix> 0-9`|直接跳到第 N 個 window|
|`<Prefix> w`|列出所有 window 上下選|
|`<Prefix> l`|切回剛剛那個 window|
|`<Prefix> f`|搜尋 window(輸入關鍵字找)|

### 📐 Pane 操作(分割視窗)

|快捷鍵|功能|
|---|---|
|`<Prefix> %`|**左右**分割 pane|
|`<Prefix> "`|**上下**分割 pane|
|`<Prefix> 方向鍵`|移動到該方向的 pane|
|`<Prefix> o`|切換到下一個 pane|
|`<Prefix> q`|顯示 pane 編號(快速辨識)|
|`<Prefix> x`|關閉當前 pane(會問 y/n)|
|`<Prefix> z`|**Z**oom 全螢幕當前 pane / 還原(超好用)|
|`<Prefix> {`|把當前 pane 跟前一個交換|
|`<Prefix> }`|把當前 pane 跟後一個交換|
|`<Prefix> 空白鍵`|切換 pane 排列方式(預設 5 種)|
|`<Prefix> !`|把當前 pane 變成獨立 window|

### 📜 Scroll / Copy Mode

|快捷鍵|功能|
|---|---|
|`<Prefix> [`|進入 scroll/copy mode|
|`↑ ↓ PgUp PgDn`|在 mode 內捲動(進入後直接用)|
|`/`|向下搜尋(在 copy mode 內)|
|`?`|向上搜尋|
|`空白鍵`|開始選取(在 copy mode 內)|
|`Enter`|複製選取內容到 tmux buffer|
|`q`|離開 copy mode|
|`<Prefix> ]`|貼上 tmux buffer 內容|

### 🔧 雜項

|快捷鍵|功能|
|---|---|
|`<Prefix> t`|顯示**時鐘**(沒事可以炫一下)|
|`<Prefix> :`|進入 command mode(手打指令)|
|`<Prefix> r`|Reload 設定檔(需在 `.tmux.conf` 設定 bind)|
|`<Prefix> Ctrl+B`|送出真正的 `Ctrl+B` 給 shell(罕用)|

---

## 🧭 Part 3:實戰場景速查

### 場景 1:第一次建立躺著工作 session

```bash
tmux new -s claude
# 進入後執行
claude
# 派工後離開
<Prefix> d
```

### 場景 2:手機連回來看任務跑哪了

```bash
ssh kirin@host-linux-mint
tmux ls                    # 看看有哪些 session
tmux attach -t claude      # 進入
```

### 場景 3:看 Claude 之前輸出的東西(滾動回去)

```
<Prefix> [        # 進入 scroll mode
↑↑↑↑              # 捲上去
q                 # 看完離開
```

### 場景 4:同時跑前端和後端

```bash
# Window A:前端
tmux new -s frontend
npm run dev
<Prefix> d

# Window B:後端
tmux new -s backend
docker-compose up
<Prefix> d

# 隨時切換
tmux attach -t frontend
<Prefix> s        # 開 session 列表跳到 backend
```

或者用 Window:

```bash
tmux new -s dev
# 進去後
<Prefix> c        # 開新 window 跑後端
<Prefix> 0        # 回到 window 0(前端)
<Prefix> 1        # 切到 window 1(後端)
```

### 場景 5:在同個畫面同時看 log 和編輯

```bash
tmux new -s work
<Prefix> %        # 左右切兩半
# 左邊跑 claude
claude
# 切到右邊
<Prefix> →
# 右邊看 log
tail -f /var/log/syslog
# 把 log 那塊放大看
<Prefix> z        # zoom
# 看完縮回
<Prefix> z
```

### 場景 6:踢掉手機上殘留的 attach,讓筆電獨享

```bash
tmux attach -t claude -d   # -d 強制踢掉其他 client
```

### 場景 7:整理 session 命名

```
<Prefix> $        # 在當前 session 內重新命名
<Prefix> ,        # 重新命名 window
```

---

## ⚙️ Part 4:推薦的 `.tmux.conf` 設定

放在 `~/.tmux.conf`,讓 tmux 更好用:

```conf
# ── 基礎 ────────────────────────────────────────
set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",*256col*:Tc"
set -g history-limit 50000
set -g mouse on
set -sg escape-time 0

# ── Session 行為 ────────────────────────────────
set -g detach-on-destroy off
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# ── 設定檔熱重載(<Prefix> r 重載)─────────────
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# ── 直覺的分割鍵(可選)─────────────────────────
# 預設 % 和 " 不直觀,改成 | 和 -
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# ── vim 風格的 pane 移動(可選)─────────────────
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# ── 簡化 prefix(可選,Ctrl+A 比 Ctrl+B 好按)──
# unbind C-b
# set -g prefix C-a
# bind C-a send-prefix
```

修改後重載:

```bash
tmux source-file ~/.tmux.conf
# 或在 tmux 裡
<Prefix> :source-file ~/.tmux.conf
```

---

## 💡 小撇步

### 完全忘記快捷鍵時

```
<Prefix> ?
```

會列出所有 binding,按 `q` 離開。

### 想知道現在在哪個 session

底部狀態列左下角會顯示 `[session-name]`。

### 不小心按錯鍵卡住

```
Esc          # 大多數情況可解
q            # copy mode 用
Enter        # 對話框用
```

### Mouse Mode 副作用

啟用 `set -g mouse on` 後,**用滑鼠選取文字會不能直接 Cmd+C 複製**(因為 tmux 攔截了滑鼠事件)。解法:

- macOS:選取時**按住 Option 鍵**再拖,就會繞過 tmux
- Linux:**按住 Shift 鍵**再選取
- Windows Terminal:同 Linux

---

## 🚫 新手常踩的雷

|雷|解法|
|---|---|
|按 Ctrl+B 同時按 D|應該是先按 Ctrl+B **放開**,再按 D|
|在 SSH 斷線後失去進度|一開始就要 `tmux new -s xxx`,不要直接跑命令|
|多 client attach 畫面變小|用 `tmux attach -d` 踢其他 client|
|寫了 `.tmux.conf` 沒生效|要 `source-file` 或重啟 tmux server|
|ssh 斷了之後 tmux 內顏色亂掉|加 `set -g default-terminal "tmux-256color"`|
|想複製文字但選不到|啟用 mouse 後要按 Option/Shift 才能繞過|

---

## 📚 進階學習

如果你已經熟悉以上內容,進階主題:

- **tmux-resurrect**:重開機後自動恢復 session
- **tmux-continuum**:自動定期儲存
- **TPM (Tmux Plugin Manager)**:plugin 管理器
- **Powerline / Tmux Themepack**:狀態列美化
- **session 預設模板**:用腳本一鍵建立多 window 多 pane 的工作環境

這些不是必須,**先把上面 5 個必學快捷鍵內化,再考慮進階**。

---

**文件維護者**:Kirin (@surfuncle)  



