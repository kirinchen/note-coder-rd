# 把 tmux 換掉:herdr-app + herdr-core 當 agent 工作台

> VS Code、Claude 桌面版、tmux、herdr 的 TUI⋯⋯agent 散在一堆視窗裡,誰在跑、誰在等你,看不完。
> 我把整套換成 **herdr-app + herdr-core**:一個視窗,好幾台電腦上的 agent 都在裡面;想要的功能,直接請 agent 寫一個 mod。
> 這篇是影片的文字版,指令可以直接複製。

- 🎬 影片:<https://youtu.be/pLeJcFy8eRA>
- 🌐 herdr 官網:<https://herdr.dev>
- 📦 我的 mod(herdr-essentials):<https://github.com/kirinchen/herdr-essentials>
- 📺 之前那一集(tmux + herdr TUI):<https://youtu.be/C-00REk5px4>
- 📖 還沒用過 Herdr App:先看 [Herdr App 圖文入門指南](https://kirinchen.dev/notes/DevTools/Herdr-App-%E5%9C%96%E6%96%87%E5%85%A5%E9%96%80%E6%8C%87%E5%8D%97/)

不用多一筆訂閱費:agent 用的是你原本就在用的 Claude Code、Codex。

---

## 先把名字講清楚

| 名字 | 是什麼 |
|---|---|
| herdr TUI(舊版) | 在終端機裡跑的那一版,以前大家說的 herdr |
| herdr-app | 新版前面有視窗的那一塊 |
| herdr-core | 新版後面的那一塊,terminal 都住在這裡;指令是 `hc` |

新舊是兩套,**不互通**:舊版開的 agent,app 看不到;app 開的,舊版也看不到。
要用 app,後面就只裝 herdr-core,舊的 TUI 不用裝。

## 怎麼裝

### ① 你面前這台:裝 herdr-app

```sh
curl -fsSL https://releases.herdr.dev/app/install.sh | sh
```

這台還沒有 herdr-core 的話,它會順便裝好。

### ② 不需要畫面的機器(server):只裝 herdr-core

```sh
curl -fsSL https://releases.herdr.dev/core/install.sh | sh
```

### ③ 回到你這台,把那台加進來

```sh
hc machine add <機器名>
```

前提只有一個:從這台 `ssh <機器名>` 要能直接進去,不問密碼、不問任何問題。

加進來之後,每個 agent 都有一個位址:`機器名/編號`,例如 `server/t29`。要叫它做事:

```sh
hc send server/t29 "跑一下測試"
```

### 手機

官方有手機 app(iPhone / iPad / Android):<https://herdr.dev/mobile>。我還沒實際用過,只知道有這個選擇。

## 接不起來的時候

| 看到什麼 | 原因 | 怎麼修 |
|---|---|---|
| `Connection timed out` | ssh 設定檔的 IP 是舊的 | 改 `~/.ssh/config` 的 `HostName` |
| `Host key verification failed` | 沒手動 ssh 進去過 | 先 `ssh <機器名>` 一次,答 yes |
| `wants a login typed by a person` | 公鑰還沒放上去 | `ssh-copy-id <機器名>` |
| 找不到 `hc` | 那台的 PATH 沒包含 | PATH 加 `~/.local/bin` |

懶得自己查:把錯誤訊息貼給你的 agent,請它幫你修。

## 什麼是 mod

herdr-app 整個畫面是用 mod 組出來的:側邊欄是 mod,tab 列是 mod,連設定頁都是。
一個 mod 就是一個資料夾,裡面一支 `main.js`。存檔,畫面馬上換。

所以想要什麼功能,不用等官方:請 agent 寫一個 mod 就有了。
工具會散,是因為每一套的介面各管各的;介面可以自己改成想要的樣子,就不用為了一個順手的畫面再多裝一套。

⚠️ 內建的 mod 每次啟動都會被蓋回去。不要直接改它,複製一份、換個名字再改。

## 我的 mod:herdr-essentials

```sh
git clone https://github.com/kirinchen/herdr-essentials.git
cd herdr-essentials
./install.sh
herdr-app do layout.switch --name k-main
```

- 回原本的版面:`herdr-app do layout.switch --name main`
- **Windows**:先開「開發人員模式」再跑 `install.sh`(mod 是用 symlink 裝的)。
- **Windows 改 mod 不會馬上換**:要切一次版面才會重新載入,先切到 `main` 再切回 `k-main`。

影片裡出現的幾個:

| mod | 做什麼 |
|---|---|
| `k-tabs` | tab 的 × 改成「收起」:tab 不見,程式繼續跑;🗑 才是真的關 |
| `k-spaces` | 點 Space 名字,底下的 tab 展開 / 收起 |
| `k-pick` | `ctrl-f12` 點畫面上任何一塊,把「是哪一塊」複製給 agent |
| `k-agents` | 圖示換成模型(寶石 Opus、羽毛 Sonnet、葉子 Haiku),底色越深開得越強 |
| `k-chat` | 幾個 agent 開會:主席點名發言,談定了才准一個動手 |
| `k-float` / `k-note` | 桌面便利貼:herdr 縮小了它還在,agent 寫一行進檔案它就多一行 |

完整清單和說明在 repo 的 README。

## 換過來之後要注意的四件事

1. **在 herdr-app 關 tab = 結束裡面的程式。** 跟 tmux 不一樣(tmux 離開畫面,程式還在)。所以我才把 × 改成收起。
2. **別隨便重啟 herdr-core。** 所有 agent 都住在裡面,一重啟全部一起受影響。
3. **用指令開的 terminal,要寫進 app 的排版,app 才看得到**;而且要寫到程式所在的那一台。
4. **Windows**:同一個 profile 不要開兩個視窗,第二個會搶走第一個的控制通道(畫面正常,指令全部打不到)。要多開視窗就另外加一個 profile。
