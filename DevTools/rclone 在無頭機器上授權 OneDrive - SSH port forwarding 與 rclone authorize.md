---
title: rclone 在無頭機器上授權 OneDrive — SSH port forwarding 與 rclone authorize
tags: [rclone, ssh, onedrive, backup, headless]

---

# rclone 在無頭機器上授權 OneDrive

> **情境**:一台只能 SSH 進去的 Linux 機器(沒有桌面 / 螢幕沒訊號 / 純 server),
> 要用 rclone 把某個目錄備份到 OneDrive。而 rclone 的 OAuth 授權**需要瀏覽器**。
>
> 這篇記的是實際踩過一輪的流程,包含三個文件上不會寫、但一定會撞到的坑。

---

## 0. 為什麼不是「裝個 OneDrive app 就好」

**微軟沒有官方的 Linux OneDrive client。** 只有 Windows / macOS / iOS / Android。

Linux 上有兩條路,先講清楚差別,因為選錯的代價很大:

| | [abraunegg/onedrive](https://github.com/abraunegg/onedrive) | rclone |
|---|---|---|
| 形態 | 常駐 daemon,inotify 即時同步 | CLI,跑完就結束 |
| idle 資源 | 約 30–80 MB RSS | **0**(不跑就不存在) |
| **預設行為** | ⚠️ **把整個帳號同步下來** | 只碰你指定的那一個路徑 |
| 刪除保護 | 靠 OneDrive 回收桶 | `copy` 模式根本不刪 |

第三列是關鍵:**如果你的 OneDrive 已經有幾百 GB,abraunegg 的預設會開始把它們全部拉下來**
(要設 `sync_list` 選擇性同步才不會)。

而如果你的資料是「一批幾乎不動的靜態檔」,常駐 daemon 買不到任何東西 ——
你不會在存檔後三秒內就需要它已經上雲。這種情況 **rclone + 排程**是更小的複雜度。

---

## 1. 安裝 rclone(不需要 sudo)

發行版倉庫裡的 rclone 通常很舊(Ubuntu 24.04 是 2022 年的 1.60),而且要 sudo。
官方 binary 丟進 `~/.local/bin` 就好:

```bash
cd /tmp
curl -fsSL -o rclone.zip https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip -q rclone.zip
mkdir -p ~/.local/bin
cp rclone-v*/rclone ~/.local/bin/ && chmod +x ~/.local/bin/rclone
rclone version
```

> `~/.local/bin` 通常已經在 PATH 裡 —— Debian 系的 `~/.profile` 有這段:
> ```sh
> if [ -d "$HOME/.local/bin" ] ; then PATH="$HOME/.local/bin:$PATH" ; fi
> ```
> ⚠️ 但它是**登入時**判斷目錄存不存在。如果 `~/.local/bin` 是你剛建的,
> 要重新登入(或 `source ~/.profile`)才生效。

---

## 2. 確認你這台到底開不開得了瀏覽器

先別急著跑 `rclone config`,花 10 秒確認:

```bash
echo "DISPLAY=$DISPLAY  SSH=$SSH_CONNECTION"
loginctl list-sessions
```

`DISPLAY` 是空的 + `SSH_CONNECTION` 有值 → **無頭**,瀏覽器開不起來。
這時 `rclone config` 那題答 `y` 會卡住。

---

## 3. 方法 A:SSH port forwarding(推薦)

rclone 的授權流程是它**自己在 `127.0.0.1:53682` 開一個小 web server**,
等瀏覽器把 OAuth 的 code 送回來。所以只要把那個 port 轉發到你本機就行。

### 3.1 連線時加 `-L`

```bash
ssh -L 53682:localhost:53682 你的帳號@遠端主機
```

### 3.2 在那個 session 裡跑 config

```bash
rclone config
```

| 問你 | 答 |
|---|---|
| `n/s/q` | `n`(New remote) |
| `name>` | `onedrive`(自己取,但後面腳本要對得上) |
| `Storage>` | `onedrive`(或選單編號,**不要記死數字,版本會變**) |
| `client_id>` / `client_secret>` | Enter 留空(用 rclone 內建的) |
| `region>` | `1`(Microsoft Cloud Global) |
| `Edit advanced config?` | `n` |
| **`Use web browser to automatically authenticate?`** | **`y`** |

它會印出:

```
If your browser doesn't open automatically go to the following link:
http://127.0.0.1:53682/auth?state=xxxxx
```

**把這個網址複製到你本機的瀏覽器打開** —— 因為有 `-L`,你本機的 53682
會被轉發到遠端那支 rclone。登入授權完,遠端就收到 code 了。

### 3.3 GUI 終端機怎麼設(以 Tabby 為例)

不想每次手打 `-L` 的話,終端機工具通常有內建設定。Tabby:

**Settings → Profiles & connections → 編輯該 SSH profile → Port forwarding → Add**

| 欄位 | 值 |
|---|---|
| Type | **Local** |
| Local address | `127.0.0.1` |
| Local port | `53682` |
| Destination host | `localhost` |
| Destination port | `53682` |

存檔後**要重新連線**(既有 session 不會套用)。

---

## 4. 方法 B:`rclone authorize`(不想動 SSH 設定時)

rclone 內建了這個流程:授權在**另一台有瀏覽器的機器**上做,把 token 貼回來。

1. **無頭這台**:`rclone config`,`Use web browser...?` 答 **`n`**
2. 它會印出一行指令,像:
   ```
   rclone authorize "onedrive" "eyJ..."
   ```
3. **有瀏覽器那台**(Windows / Mac 都行,先裝 rclone):整行貼上跑
4. 瀏覽器自動開 → 登入 → 它吐出一大段 `{"access_token":"...","refresh_token":"..."}`
5. 把那段 JSON 貼回**無頭這台**正在等的提示 → Enter

> 代價:要在第二台裝 rclone。方法 A 不用。

---

## 5. 三個實際踩到的坑

### 坑 1:`Got code` 之後出現 `ChannelOpenFailure`,看起來像失敗

```
2026/09/10 10:22:31 NOTICE: Got code
...
SSH X Remote has rejected the forwarded connection to localhost:53682:
      Error: ChannelOpenFailure(ConnectFailed)
```

**授權已經成功了** —— `Got code` 那行就是證據。

底下那個錯誤是**授權完成之後**瀏覽器又多打了一次請求(通常是 favicon),
而 rclone 那時已經把 53682 的 listener 收掉了。**無害,忽略**。

### 坑 2:drive 選單按 Enter 會選到錯的(這個會真的失敗)

OneDrive Personal 的 `config_driveid` 選單長這樣:

```
 1 / ODCMetadataArchive (personal)         ← 預設值!
 2 / AEEE102E-CFF8-4E2A-89C6-... (personal)
 3 / Bundles_b896e2bb... (personal)
 ...
 6 / OneDrive (personal)                   ← 你要的
config_driveid>
```

**直接按 Enter 會選到第 1 個 `ODCMetadataArchive`** —— 那是 OneDrive 的內部
metadata 磁碟,不是你的檔案空間。結果:

```
Failed to query root for drive "b!m889...":
HTTP error 400: {"code":"invalidRequest","message":"ObjectHandle is Invalid"}
```

然後 rclone 會**退回上一題**(不是整個重來)。這時直接在 `config_type>`
重答一次,到 drive 那題**明確打 `6`**(標著 `OneDrive (personal)` 的那個)。

> 其餘那些 GUID 名字的都是內部 drive;`Bundles_...` 是相簿 bundle,
> 雖然 drive id 跟 `OneDrive (personal)` 一樣但語義不同。認**標籤**不要認編號。

### 坑 3:`sync` 會刪掉你目的地既有的東西

這是最貴的一個坑,雖然我沒踩(但差點)。

```bash
rclone sync ~/mydata onedrive:            # ⚠️ 少打了子資料夾
```

`sync` 的語義是「**讓目的地變得跟來源一樣**」。上面這行會為了讓
OneDrive 根目錄「跟你那 300MB 的來源一樣」,而**刪掉你其他所有檔案**。

**備份用 `copy`,不要用 `sync`。**

| | `copy` | `sync` |
|---|---|---|
| 目的地多出來的檔 | 留著 | **刪掉** |
| 打錯目的地的後果 | 多一份在錯的地方 | 💀 |

`copy` 的代價是:本機刪掉的檔,雲端會一直留著。
對「不可再生的原始素材」來說那**正是你要的** —— 備份工具不該有刪除權。

---

## 6. 包成一個腳本 + 每天自動跑

### 6.1 腳本

`~/.local/bin/mybackup`:

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC="$HOME/mydata"
DST="onedrive:mydata"              # ⚠️ 專屬子資料夾,與既有檔案隔開
RCLONE="$HOME/.local/bin/rclone"
LOG="$HOME/.local/state/mybackup.log"

mkdir -p "$(dirname "$LOG")"
[ -d "$SRC" ] || { echo "✘ $SRC 不存在"; exit 1; }
# remote 沒設定就明確失敗 —— 不要靜默跑空
"$RCLONE" listremotes 2>/dev/null | grep -q '^onedrive:' || {
  echo "✘ 還沒設定 onedrive remote —— 先跑:$RCLONE config"; exit 1; }

echo "=== $(date -Is) ===" >> "$LOG"
"$RCLONE" copy "$SRC" "$DST" \
  --transfers 4 --checkers 8 \
  --log-file "$LOG" --log-level INFO --stats-one-line --stats 30s "$@"
echo "✓ $(date -Is) 備份完成 → $DST" | tee -a "$LOG"
```

```bash
chmod +x ~/.local/bin/mybackup
```

三個刻意的設計:

1. **`copy` 不是 `sync`** —— 見坑 3
2. **專屬子資料夾** `onedrive:mydata` —— 跟既有檔案完全隔開
3. **remote 沒設定就 `exit 1`** —— 不要讓它靜默跑一個空的備份然後回報成功

### 6.2 systemd user timer

`~/.config/systemd/user/mybackup.service`:

```ini
[Unit]
Description=mydata → OneDrive 備份(rclone copy,只增不刪)

[Service]
Type=oneshot
ExecStart=%h/.local/bin/mybackup
Nice=10
IOSchedulingClass=idle
```

`~/.config/systemd/user/mybackup.timer`:

```ini
[Unit]
Description=每天備份 mydata 到 OneDrive

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=30m

[Install]
WantedBy=timers.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now mybackup.timer
systemctl --user list-timers mybackup.timer
```

> ⚠️ **user timer 預設在你登出後就不跑了。** 要讓它一直跑:
> ```bash
> sudo loginctl enable-linger $USER
> loginctl show-user $USER --property=Linger   # 應該回 Linger=yes
> ```

`Nice=10` + `IOSchedulingClass=idle` 讓它不跟你搶資源 —— 備份是背景工作,
不該讓你的編譯變慢。

---

## 7. 驗證:別只看「成功」兩個字

第一次跑完一定要真的驗一次,而且**要比 hash 不是比大小**:

```bash
# 先看它打算做什麼,一個檔都不動
mybackup --dry-run

# 真的跑
mybackup

# 逐檔比 hash
rclone check ~/mydata onedrive:mydata
```

```
NOTICE: OneDrive root 'mydata': 0 differences found
NOTICE: OneDrive root 'mydata': 49 matching files
```

看到 **`0 differences found`** 才算數。`rclone size` 只比總量,
檔案內容壞掉它看不出來。

---

## 8. 一句話總結

| 問題 | 答案 |
|---|---|
| Linux 沒有官方 OneDrive app | 用 rclone |
| 無頭機器沒瀏覽器 | SSH `-L 53682:localhost:53682`,授權頁在本機開 |
| 雲端已有幾百 GB 會不會被拉下來 | rclone **不會**(只碰指定路徑);abraunegg **會** |
| 備份工具用哪個模式 | **`copy`**,不要 `sync` |
| 怎麼確認真的備份好了 | `rclone check`(比 hash),不是 `rclone size` |

---

###### tags: `rclone` `ssh` `onedrive` `backup` `headless`
