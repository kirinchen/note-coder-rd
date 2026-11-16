# tmux 與 herdr 指令、快捷鍵對照表

> 配套文件：[[tmux 速查表]]（System/Linux）、[[tmux + SSH 讓 claude 重新連接]]
> 適用版本：tmux 3.x / **herdr-app 0.1.16 + herdr-core (`hc`) 0.1.21**
> 對照依據：本機 `~/.config/herdr-app/keys.json`（未改過，等於出廠預設）、`hc --help`、`herdr-app commands`、`~/.config/herdr-app/AGENTS.md`

---

## 🎯 先搞清楚：你在用哪個 herdr？

**herdr 有兩個長得很像、但快捷鍵系統完全不同的產品**，查資料前先確認，不然會查到一堆不能用的快捷鍵：

| | **herdr（TUI 版）** | **herdr-app（原生 App 版）** ← 本文對照這個 |
|---|---|---|
| 長相 | 跟 tmux 一樣跑在終端機裡 | 獨立視窗的桌面 App（GPUI 寫的） |
| 執行檔 | `herdr` | `herdr-app` + `herdr-core` + `hc` |
| 快捷鍵模型 | **有 Prefix**，預設 `Ctrl+B`，跟 tmux 同一套邏輯 | **沒有 Prefix**，全部是 `Ctrl+Shift+X` / `Alt+X` 直接按 |
| 設定檔 | `~/.config/herdr/config.toml` | `~/.config/herdr-app/keys.json` |
| CLI | `herdr session / workspace / tab / pane / agent ...` | `hc run / ls / attach / send / wait ...` |
| 怎麼判斷 | `which herdr` 有東西 | 環境變數有 `HC_TERMINAL_SOCKET`，PATH 有 `Programs/herdr-app` |

網路上大部分「tmux vs herdr」文章寫的是 **TUI 版**（`prefix+v` 分割、`prefix+q` detach 那套）。
如果你是從 herdr.dev 下載安裝、看到的是一個有側邊欄的視窗，那你用的是 **App 版**，看本文。

---

## 🧠 名詞對照（最重要的一張表）

兩邊的「session」意思完全不一樣，先把名詞對齊：

| tmux | herdr-app | 說明 |
|---|---|---|
| **Session** | **Space** | 一個工作區，通常 = 一個 repo。沒命名的 Space 會自動跟著第一個終端的資料夾 / repo 名字走 |
| **Window** | **Tab** | Space 裡的一片工作，含多個分割的終端 |
| **Pane** | **Pane** | 分割出來的一格，可以是終端，也可以是 mod（markdown viewer、git panel） |
| 跑在 pane 裡的 process | **Terminal**（`t7`） | herdr-core 管的一個長生命週期 process，有自己的地址 `t7`，可被 `hc` 操作 |
| （無） | **Agent** | 透過 OSC 7501 回報狀態的 Terminal：`working` / `blocked` / `idle` / `done` |
| （無） | **Machine** | SSH 連過去的另一台機器，terminal 地址變成 `vps/t7` |
| tmux server | **herdr-core** | 讓 terminal 在 App 關掉後還活著的 daemon |
| `tmux` client | **herdr-app**（或 `hc attach`） | 看畫面的那一端 |

**心智模型差異**：tmux 是「session 擁有 process，client attach 進去看」。
herdr-app 是「herdr-core 擁有每一個 terminal (`t7`)，App 只是其中一種 view，Space / Tab / Pane 是排版」。
所以 App 整個關掉，`hc ls` 還看得到所有 terminal 在跑。

---

## 📦 Part 1：CLI 指令對照（在終端機打的）

### Session / Terminal 生命週期

| 要做的事 | tmux | herdr-app (`hc`) |
|---|---|---|
| 開一個新的、會一直活著的工作 | `tmux new -s claude` | `hc run --bg --name claude claude` |
| 開並直接跑指令 | `tmux new -s build "npm run dev"` | `hc run --bg --name build "npm run dev"` |
| 開並**等它跑完**（像 shell 一樣） | （無，tmux 不等） | `hc run "npm test"`（預設會 wait） |
| 列出所有 | `tmux ls` | `hc ls`（加 `--all` 含已結束） |
| 看單一個的詳細資訊 | `tmux display -t x` | `hc get t3` |
| attach 進去 | `tmux attach -t claude` | `hc attach claude`（或 `hc attach t3`） |
| 只看不碰（observe） | `tmux attach -r` | `hc attach t3 --observe` |
| attach 時先補歷史畫面 | （本來就有） | `hc attach t3 --history 200` |
| detach | `<Prefix> d` | **`Ctrl+\`**（在 `hc attach` 裡） |
| 殺掉 | `tmux kill-session -t x` | `hc kill t3`（結束 program）→ `hc rm t3`（移除紀錄） |
| 殺掉所有 | `tmux kill-server` | `hc server stop`（會把 kept terminal 停放，下次 core 起來給 shell） |
| 互動式挑一個 attach | `tmux attach` + `<Prefix> s` | 直接打 `hc`（跳選單，最上面是「new…」） |
| 在另一台機器上開 | `ssh host tmux new ...` | `hc run --on vps "cd app && npm test"` |

### 對 process 送東西 / 讀東西（自動化腳本最常用）

| 要做的事 | tmux | herdr-app (`hc`) |
|---|---|---|
| 貼文字 + Enter | `tmux send-keys -t x "text" Enter` | `hc send t3 "text"` |
| 按特殊鍵 | `tmux send-keys -t x C-c` | `hc keys t3 ctrl+c`（`enter`、`esc`、`up`…） |
| 送 Ctrl+C 並回報結果 | `tmux send-keys C-c` | `hc interrupt t3` |
| 抓畫面內容 | `tmux capture-pane -p -t x` | `hc read t3`（含 scrollback，`--lines 50`） |
| 等它跑完 / 等它印出某字 | （無，要自己 while loop） | `hc wait t3` / `hc wait t3 --output "Done"` |
| 等子 agent 全部結束 | （無） | `hc wait t3 --children all` |
| 丟檔案給它 | `tmux load-buffer` + paste | `hc put t3 ./file.txt` |
| 在遠端跑一次性指令拿輸出 | `ssh host cmd` | `hc exec --on vps git status` |
| 遠端讀寫檔 | `scp` | `hc file read\|write\|edit --on vps <path>` |
| 設定 metadata | `tmux set -t x @k v` | `hc label t3 k=v` |
| 即時監看所有變化 | （無） | `hc events` |

**`hc send` 比 `send-keys` 聰明的地方**：有 `--if idle` / `--if working` 可以限定「agent 閒著才送」，避免打斷正在工作的 Claude。

### 操作版面（Space / Tab / Pane）

tmux 的 `new-window` / `split-window` 這類指令，在 herdr-app 走 **`herdr-app do <command>`**，不是 `hc`：

| 要做的事 | tmux | herdr-app |
|---|---|---|
| 新 session / space | `tmux new -s x` | `herdr-app do space.new --name x --folder ~/proj` |
| 新 window / tab 並跑 agent | `tmux new-window -n x "claude"` | `herdr-app do tab.new --name "fix login" --argv '["claude"]' --task "修登入 bug"` |
| 分割 | `tmux split-window -h` | `herdr-app do pane.split:right`（`:down` 為上下） |
| 列出全部指令 | `tmux list-commands` | `herdr-app commands` |
| 看目前視窗結構 | `tmux list-panes -a` | `herdr-app ls` / `herdr-app status` |
| 截圖 | （無） | `herdr-app shot out.png` |

---

## ⌨️ Part 2：快捷鍵對照（在畫面裡按的）

> tmux 欄的 `<Prefix>` = 先按 `Ctrl+B` 放開再按下一鍵。
> herdr-app **沒有 Prefix**，表內的鍵就是直接同時按。
> herdr-app 欄後面括號是 keys.json 裡對應的 command 名稱，改鍵時會用到。

### 🔑 最常用的 5 個

| 功能 | tmux | herdr-app |
|---|---|---|
| Detach 離開（裡面繼續跑） | `<Prefix> d` | **直接關 App 視窗**（herdr-core 讓一切繼續跑）；`hc attach` 中則是 `Ctrl+\` |
| 列出 / 切換 session (space) | `<Prefix> s` | `Alt+1`～`Alt+9` 直跳；或 `Ctrl+Shift+B` 開側邊欄點 |
| 捲動看歷史輸出 | `<Prefix> [` 進 copy mode | **直接滑鼠滾輪 / 觸控板**，沒有 copy mode |
| 顯示所有快捷鍵 | `<Prefix> ?` | `Ctrl+?`（或 `Ctrl+Shift+/`）(`app.help`) |
| 手打指令 | `<Prefix> :` | `Ctrl+Shift+P` 命令面板 (`app.palette`) |

### 🪟 Session ↔ Space

| 功能 | tmux | herdr-app |
|---|---|---|
| 新建 | `tmux new -s x`（CLI） | `Ctrl+Shift+N` (`space.new`) |
| 重新命名 | `<Prefix> $` | `Shift+F2` (`space.rename`) |
| 列表選擇 | `<Prefix> s` | 側邊欄（`Ctrl+Shift+B` 切換顯示） |
| 跳到第 N 個 | （無預設） | `Alt+1`～`Alt+9` (`space.select:N`) |
| 上一個 / 下一個 | `<Prefix> (` / `<Prefix> )` | `Ctrl+Shift+PageUp` / `Ctrl+Shift+PageDown` (`space.prev/next`) |
| 切回剛剛那個 | `<Prefix> L` | （無預設鍵） |
| 關閉 | `tmux kill-session` | （無預設鍵，命令面板 → `space.close`，會問確認） |

### 🪟 Window ↔ Tab

| 功能 | tmux | herdr-app |
|---|---|---|
| 新建 | `<Prefix> c` | `Ctrl+Shift+T` (`tab.new`) |
| 重新命名 | `<Prefix> ,` | `F2` (`tab.rename`) |
| 關閉 | `<Prefix> &` | `Ctrl+Shift+W` (`tab.close`) |
| 下一個 / 上一個 | `<Prefix> n` / `<Prefix> p` | `Ctrl+Tab` / `Ctrl+Shift+Tab`，或 `Ctrl+PageDown` / `Ctrl+PageUp`，或 `Alt+]` / `Alt+[` (`tab.next/prev`) |
| 跳到第 N 個 | `<Prefix> 0-9` | `Ctrl+1`～`Ctrl+9` (`tab.select:N`) |
| 列表選擇 | `<Prefix> w` | 側邊欄 |
| 切回剛剛那個 | `<Prefix> l` | （無預設鍵） |
| 搜尋 window | `<Prefix> f` | `Ctrl+Shift+P` 命令面板打名字 |

### 📐 Pane

| 功能 | tmux | herdr-app |
|---|---|---|
| **左右**分割 | `<Prefix> %` | `Ctrl+Shift+O` (`pane.split:right`) |
| **上下**分割 | `<Prefix> "` | `Ctrl+Shift+E` (`pane.split:down`) |
| 移動焦點 | `<Prefix> 方向鍵` | `Alt+H/J/K/L`，或 `Alt+←` / `Alt+→` (`pane.go:*`) |
| 下一個 pane | `<Prefix> o` | （無，用 Alt+HJKL） |
| 顯示 pane 編號 | `<Prefix> q` | （無，pane 有標題列） |
| 關閉 pane | `<Prefix> x` | `Ctrl+Shift+X` (`pane.close`) |
| Zoom 全螢幕 / 還原 | `<Prefix> z` | `Ctrl+Shift+Z` (`pane.zoom`) |
| 交換 pane | `<Prefix> {` / `<Prefix> }` | `Alt+Shift+H/J/K/L` (`pane.swap:*`)，或滑鼠拖拉 pane 到另一個的中央 |
| 切換排列方式 | `<Prefix> 空白鍵` | `Ctrl+Shift+L` 切換 layout (`layout.switch`) |
| 把 pane 變獨立 window | `<Prefix> !` | 滑鼠把 pane 拖到 tab 列 |
| 調整大小 | `<Prefix> Ctrl+方向鍵` | 滑鼠拖分隔線（無預設鍵） |

### 🤖 Agent 導航（herdr 獨有，tmux 沒有對應）

| 功能 | tmux | herdr-app |
|---|---|---|
| 跳到下一個 / 上一個 agent | （無） | `Alt+↓` / `Alt+↑` (`agent.next/prev`) |
| 跳到特定 agent | （無） | 命令面板 → `agent.focus` |
| 看哪個 agent 卡住等你 | （無） | 側邊欄顏色：Space / Tab 的狀態 = 裡面最緊急 agent 的狀態（`blocked` > `done` > `working` > `idle`） |

### 📜 Scroll / Copy / Search

| 功能 | tmux | herdr-app |
|---|---|---|
| 進 scroll mode | `<Prefix> [` | **沒有 mode**，滑鼠滾輪直接捲 |
| 搜尋 | copy mode 內 `/` `?` | `Ctrl+Shift+F` 開 find bar (`terminal.search`)；Enter / Shift+Enter 下一個 / 上一個；Esc 關 |
| 選取 | copy mode 內 `空白鍵` | 滑鼠拖拉（拖過邊界會自動捲） |
| 複製 | copy mode 內 `Enter` | `Ctrl+Shift+C` 或 `Ctrl+Insert` (`terminal.copy`) |
| 貼上 | `<Prefix> ]` | `Ctrl+Shift+V` 或 `Shift+Insert` (`terminal.paste`) |
| 貼圖片 | （無） | `Ctrl+V` (`terminal.paste_image`；Windows 上退化為貼文字) |
| 全選 | （無） | （無預設鍵，`terminal.select_all`；macOS 用 Cmd+A） |
| 離開 mode | `q` | （不需要） |

### 🔧 雜項

| 功能 | tmux | herdr-app |
|---|---|---|
| 顯示所有 binding | `<Prefix> ?` | `Ctrl+?` / `Ctrl+Shift+/` / `Ctrl+Shift+?` (`app.help`) |
| 命令模式 | `<Prefix> :` | `Ctrl+Shift+P` (`app.palette`) |
| 設定 | 編輯 `.tmux.conf` | `Ctrl+,` (`app.settings`) |
| Reload 設定 | `<Prefix> r`（需自綁） | **不用**，`keys.json` / `settings.json` 存檔即生效（live） |
| 側邊欄 | （無） | `Ctrl+Shift+B` (`layout.toggle:sidebar`) |
| mod / 面板選擇器 | （無） | `Ctrl+Shift+K` (`mod.open:picker`) |
| 字體縮放 | 終端機自己的 | `Ctrl+=` / `Ctrl+-` / `Ctrl+0` (`app.zoom_in/out/reset`) |
| 送出真正的 Ctrl+B | `<Prefix> Ctrl+B` | 不需要，沒 prefix |
| **救命鍵**（版面 / 按鍵壞了） | `tmux kill-server` | **`Ctrl+Alt+Shift+Esc`**：把 layout 與 keys.json 備份成 `*.reset-<time>` 後回到預設，**不可被重綁** |

### 終端機內的文字編輯鍵（herdr-app 預設幫你翻譯）

| 按鍵 | 送給 shell 的 | 效果 |
|---|---|---|
| `Ctrl+Backspace` | `\u0017` (Ctrl+W) | 刪前一個字 |
| `Ctrl+←` | `\u001bb` (Alt+B) | 游標往前一個字 |
| `Ctrl+→` | `\u001bf` (Alt+F) | 游標往後一個字 |

---

## 🧭 Part 3：實戰場景對照

### 場景 1：開一個躺著跑的 Claude，派工後離開

```bash
# tmux
tmux new -s claude
claude
<Prefix> d

# herdr-app（純 CLI，不開視窗）
hc run --bg --name claude claude
# 在 App 裡：Ctrl+Shift+T 開 tab 跑 claude，然後直接關視窗或切去別的 space
```

### 場景 2：回來看它跑到哪

```bash
# tmux
tmux ls
tmux attach -t claude

# herdr-app
hc ls                        # 看狀態欄：working / blocked / idle / done
hc read claude --lines 50    # 不 attach 直接看最後 50 行
hc attach claude             # 真的要進去；Ctrl+\ 離開
```

### 場景 3：等 agent 做完再做下一步（腳本）

```bash
# tmux：沒有內建 wait，要自己 sleep + capture-pane 比對
# herdr-app：
hc send claude "幫我把測試修好" --if idle
hc wait claude                       # 一直等到它 idle / blocked / done
hc read claude --lines 20
```

### 場景 4：同時跑前端和後端

```bash
# tmux
tmux new -s dev
<Prefix> c   # window 1 跑後端
<Prefix> 0   # 回 window 0

# herdr-app（兩個 tab）
Ctrl+Shift+T  → 第二個 tab 跑後端
Ctrl+1 / Ctrl+2 切換

# 或純 CLI
hc run --bg --name fe "npm run dev"
hc run --bg --name be "docker compose up"
```

### 場景 5：左邊 Claude、右邊 log

```
# tmux
<Prefix> %   →   <Prefix> →   →   tail -f ...   →   <Prefix> z 放大

# herdr-app
Ctrl+Shift+O →  Alt+L  →  tail -f ...  →  Ctrl+Shift+Z 放大 / 還原
```

### 場景 6：多個 agent，哪個在等我？

```
# tmux：一個一個 <Prefix> s 切進去看
# herdr-app：
Alt+↓ / Alt+↑        在 agent 之間跳
Ctrl+Shift+B         開側邊欄，紅色（blocked）的就是在等你
```

### 場景 7：操作遠端機器

```bash
# tmux
ssh vps
tmux attach -t work

# herdr-app
hc machine add vps                     # 一次性
hc run --on vps --bg --name work "cd app && claude"
hc attach vps/work                     # 地址帶 machine/ 前綴
hc exec --on vps git status            # 單發指令拿輸出
```

---

## ⚙️ Part 4：改快捷鍵

### tmux：`~/.tmux.conf`

```conf
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
```

改完要 `tmux source-file ~/.tmux.conf`。

### herdr-app：`~/.config/herdr-app/keys.json`

Windows 也是 `C:\Users\<you>\.config\herdr-app\keys.json`（不在 `%APPDATA%`）。存檔即生效。

```json
{
  "global": {
    "ctrl-shift-o": "pane.split:right",
    "alt-1": "space.select:1",
    "ctrl-shift-r": { "command": "layout.resize", "args": { "id": "sidebar", "px": 240 } }
  },
  "terminal": {
    "ctrl-shift-c": "terminal.copy",
    "ctrl-1": "terminal"
  },
  "layout:terminals": { "ctrl-shift-pagedown": "space.next" },
  "mod:editor": { "ctrl-s": "editor.save" }
}
```

規則：

- **四種 section**：`global` → `layout:<name>`（該 layout 顯示時）→ `terminal`（終端有焦點時）→ `mod:<name>`。愈特定愈優先。
- **四種 action**：
  - 指令名 `tab.new`，或帶第一個參數 `pane.split:right`
  - `send:<text>`：把文字打進終端（上面 `Ctrl+←` 就是這樣做的）
  - `<mod>.<command>`：丟給某個 mod
  - `"terminal"`：**把這個鍵讓回給程式**（例如上例 `ctrl-1` 不再切 tab，vim 可以用）
- 多參數用物件：`{"command": ..., "args": {...}}`
- 鍵名用 `-` 連接小寫：`ctrl-shift-t`、`alt-]`、`shift-f2`

### ⚠️ herdr-app 改鍵的兩個雷

| 雷 | 說明 |
|---|---|
| **改過一次就「整份是你的」** | 沒改過的 keys.json 升版會自動拿到新預設鍵。**只要你動過任何一個字元（連空白都算）**，之後新版的新快捷鍵就不會自動加進來，要自己手動補。 |
| **不要綁走 shell 靠的 Ctrl 鍵** | 沒綁的 `Ctrl+字母` 會直接送給程式。別自己去綁 `ctrl-c / d / l / r / w / a / e / u / k / z / [`，會把 shell、vim、Claude Code 的基本操作吃掉。Windows 上 `Ctrl+Alt` = AltGr，也別碰。 |

JSON 寫壞了不會爆：App 保留上一份可用版本，並寫出 `keys.error` 告訴你哪裡錯。

---

## 🚫 從 tmux 過來常踩的雷

| 雷 | 解法 |
|---|---|
| 按 `Ctrl+B` 等 prefix，結果送進 shell 了 | herdr-app **沒有 prefix**，直接按 `Ctrl+Shift+X` 系列 |
| 找 `<Prefix> [` 的 copy mode 找不到 | 沒有 copy mode，滑鼠滾輪捲、`Ctrl+Shift+F` 搜、拖拉選取 `Ctrl+Shift+C` 複製 |
| 找 detach 鍵 | App 版：關視窗就好，東西都在 herdr-core 裡。`hc attach` 裡才是 `Ctrl+\` |
| `hc kill` 完 `hc ls` 還看得到 | `kill` 只結束 program，紀錄要 `hc rm`（或 `hc ls` 不加 `--all` 就看不到了） |
| 想用 `herdr session list` | 那是 TUI 版指令，App 版是 `hc ls`。查到 `prefix+v` / `prefix+q` 的文章也都是 TUI 版 |
| 想在腳本裡開 tab / split | 不是 `hc`，是 `herdr-app do tab.new ...` / `herdr-app do pane.split:right` |
| 改了 keys.json 新版沒拿到新鍵 | 見上面「改過一次就整份是你的」；用 `Ctrl+Alt+Shift+Esc` 可以 reset 回預設（舊檔會備份） |
| `Ctrl+Tab` 在 Windows 被別的東西吃掉 | 改用 `Ctrl+PageDown` 或 `Alt+]`，三組預設都綁 `tab.next` |

---

## 📚 延伸

- `hc --skill`：印出給 AI agent 看的完整 herdr-core 操作手冊（含怎麼用 `hc` 協調多個 sub-agent）
- `herdr-app --skill`：印出怎麼用 `herdr-app do` 驅動視窗、寫 layout 與 mod
- `~/.config/herdr-app/AGENTS.md`：本機完整文件，Space / Tab / Pane 模型、layout、mod、keys 全在裡面
- 官方：[herdr.dev](https://herdr.dev)、[herdrdev/herdr-app](https://github.com/herdrdev/herdr-app)、[herdrdev/herdr-core](https://github.com/herdrdev/herdr-core)
- TUI 版（另一個產品，別混）：[herdrdev/herdr](https://github.com/herdrdev/herdr)

---

**文件維護者**：Kirin (@surfuncle)
