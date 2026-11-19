# herdr-app 加 machine 失敗：「it needs you to answer something」

> 配套文件：[[tmux 與 herdr 指令、快捷鍵對照表]]（Server 要裝什麼、本機 App 怎麼接過去）
> 環境：Windows 11 + herdr-app 0.1.16 / hc 0.1.21，server 是 Linux Mint，走 Tailscale（100.x.x.x）

---

## 🧨 症狀

在 herdr-app 側邊欄用「Add machine」輸入 `kirin@100.114.93.81`，跳出：

```
Couldn't add kirin@100.114.93.81 here: it needs you to answer something
(a password, a new host key, or installing herdr-core there).
Run `hc machine add kirin@100.114.93.81` in a terminal.
`hc machine --help` shows the options.
```

幾乎每個第一次接新機器的人都會撞到，因為它不是 bug，是**設計如此**。

---

## 🧠 為什麼

`hc machine add` 底層就是開一條 **ssh** 到那台機器，把本機的 `hc` 複製過去並啟動 herdr-core。
App 的 GUI 是用**非互動**方式跑這條 ssh：沒有終端機、沒有人可以打字。只要 ssh 中途需要你「回答任何一個問題」，GUI 就只能放棄，叫你去終端機跑。

會要你回答的，就是錯誤訊息列的三件事：

| ssh 會問的 | 什麼情況 | 怎麼判斷 |
|---|---|---|
| **新 host key** | 這台機器第一次連，`~/.ssh/known_hosts` 沒有它的指紋，ssh 要你答 `yes` | `ssh -o BatchMode=yes user@host true` 噴 `Host key verification failed.` |
| **密碼** | 本機沒有 SSH 金鑰、或公鑰沒放到 server 的 `authorized_keys`，只能用密碼登入 | 同上指令噴 `Permission denied (publickey,password).` |
| **裝 herdr-core** | server 上沒有 `hc`，或版本太舊，`hc` 要問你「要不要複製過去？」 | 前兩項都過了還是失敗 |

**三個條件全部要免互動**，GUI 才接得上。第一次接新機器幾乎一定三個都中。

---

## 🔧 修法（一次做完，之後 GUI 就能直接加）

### Step 0：先確認卡在哪

```bash
ssh -o BatchMode=yes -o ConnectTimeout=8 kirin@100.114.93.81 'echo SSH_OK'
```

`BatchMode=yes` 就是「禁止任何互動」，等同 GUI 的行為。看它噴什麼就知道卡哪一關。

### Step 1：收 host key（解決「新 host key」）

```bash
ssh-keyscan -T 8 100.114.93.81 >> ~/.ssh/known_hosts
```

> 嚴格來說 `ssh-keyscan` 沒有驗證指紋，跟手動 ssh 按 `yes` 一樣信任第一次連到的那台。內網 / Tailscale 上可接受；公網機器就先 `ssh` 進去比對一次指紋。

### Step 2：產生金鑰（如果 `~/.ssh` 裡沒有 `id_*` 檔）

```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519 -C "kirin-windows-herdr"
```

`-N ""` 是空 passphrase。**不能設 passphrase**，否則 ssh 會問你 passphrase，又變成互動（除非你有跑 ssh-agent）。

### Step 3：把公鑰放到 server（解決「密碼」，這步要打一次密碼）

Windows 沒有 `ssh-copy-id`，手動做同一件事。在 **PowerShell**：

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh kirin@100.114.93.81 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

或在 Git Bash：

```bash
cat ~/.ssh/id_ed25519.pub | ssh kirin@100.114.93.81 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

這是**整個流程唯一一次**要輸入密碼的地方。做完再跑 Step 0，應該印 `SSH_OK`。

### Step 4：在終端機加 machine，順便把 herdr-core 裝過去（解決「裝 herdr-core」）

```bash
hc machine add kirin@100.114.93.81 --name mint --install
```

- `--install`：server 沒有 `hc` 或太舊時直接複製過去，不問。
- `--name mint`：之後地址是 `mint/t7`，比 `100.114.93.81/t7` 好打。

成功後：

```bash
hc machine ls                          # 看得到 mint
hc exec --on mint hostname             # 單發指令驗證
hc run --on mint --bg --name claude claude
hc attach mint/claude                  # Ctrl+\ 離開
```

herdr-app 側邊欄會自動多出 `mint` 區塊，之後在上面開 Space 就住在 server 上。

---

## ⚡ 懶人版：四行搞定（第三行會問一次密碼）

```bash
ssh-keyscan -T 8 100.114.93.81 >> ~/.ssh/known_hosts
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519 -C "herdr"
cat ~/.ssh/id_ed25519.pub | ssh kirin@100.114.93.81 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
hc machine add kirin@100.114.93.81 --name mint --install
```

---

## 💡 讓之後更順的小事

### 用 `~/.ssh/config` 給它一個名字

```
Host mint
    HostName 100.114.93.81
    User kirin
    IdentityFile ~/.ssh/id_ed25519
```

之後 `ssh mint`、`hc machine add mint` 都能用，GUI 的 Add machine 也能直接填 `mint`。

### 已經加過、某天突然又跳「needs you to answer」

| 可能原因 | 解法 |
|---|---|
| server 重灌 / 換 IP，host key 變了 | `ssh-keygen -R 100.114.93.81` 刪舊指紋，再 Step 1 |
| 本機 `hc` 升版了，server 的太舊 | `hc machine update mint`，terminal 不會斷 |
| server 端 `authorized_keys` 權限被改壞 | `chmod 700 ~/.ssh; chmod 600 ~/.ssh/authorized_keys` |
| 密碼 / 2FA 類 | `hc machine login mint` 在終端機補登入一次 |

### Tailscale 使用者

Host 填 Tailscale IP（100.x）或 MagicDNS 名字都行。如果 server 有開 **Tailscale SSH**，認證走 Tailscale 身分，Step 2 / 3 可以跳過，但 Step 1 的 host key 還是要收。

---

## 🚫 常見錯誤

| 錯 | 為什麼 |
|---|---|
| 以為是 Tailscale 不通 | 不是，`ssh` 連得到才會問你 host key。真正不通會是 timeout |
| 金鑰設了 passphrase | ssh 會問 passphrase → 又是互動 → GUI 一樣失敗。要空 passphrase 或跑 ssh-agent |
| 在 server 上先手動裝 herdr-core | 不用，`--install` 會把**本機這支** `hc` 複製過去，版本才會一致 |
| 只做了 Step 3 沒做 Step 1 | 密碼關過了但 host key 關還在，一樣失敗。`BatchMode=yes` 測一次最準 |
| 用 `hc machine add` 時沒加 `--install`，在 GUI 又失敗 | 第一次它會在終端機問你「要不要裝？」，你在終端機答 yes 就好；但 GUI 永遠答不了 |

---

**文件維護者**：Kirin (@surfuncle)
