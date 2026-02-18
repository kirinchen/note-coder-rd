

# [DevOps] Linux Mint 遠端桌面雙雄：RustDesk 與 NoMachine 安裝攻略 (含防火牆設定)

**作者：Kirin Chen**

**環境：Linux Mint (MATE/Cinnamon) / GPD MicroPC**

## 前言：為什麼要拋棄 Chrome Remote Desktop？

對於擁有 GPD MicroPC 或自架 Home Lab 的開發者來說，Google Chrome Remote Desktop (CRD) 雖然方便，但它的「虛擬 Session 機制」常常導致 **"Could not acquire name on session bus"** 的登入衝突，且在內網 (LAN) 環境下，它的畫質與延遲並非頂尖。

為了追求 **「即開即用」** 與 **「極致流暢」**，這篇文章將介紹兩款最強大的替代方案：

1. **RustDesk**：開源界的 TeamViewer，設定最簡單，支援自架中繼。
    
2. **NoMachine**：區網內的效能怪獸，採用 NX 協定，畫質與反應速度接近原生。
    

---

## Part 1: RustDesk —— 開源、簡單、可穿透內網

RustDesk 是目前 GitHub 上最熱門的遠端軟體，它不需要複雜的 Port Forwarding 就能穿透防火牆 (NAT)，非常適合「人在外面，臨時要連回家」的情境。

### 1. 下載與安裝

RustDesk 提供 `.deb` 安裝包，直接去 GitHub Release 下載即可。

**Terminal 安裝流：**



```Bash
# 1. 下載最新版 (請自行確認 GitHub 上的最新版號，這裡以 1.2.3 為例)
wget https://github.com/rustdesk/rustdesk/releases/download/1.2.3/rustdesk-1.2.3-x86_64.deb

# 2. 安裝
sudo apt install ./rustdesk-1.2.3-x86_64.deb

# 3. 如果遇到依賴錯誤，執行這行修復
sudo apt --fix-broken install
```

### 2. 啟動與權限設定

安裝後，在選單搜尋 "RustDesk" 啟動。

- **左側 ID/Password**：這是你的門牌號碼，記下來。
    
- **服務啟動**：確保視窗下方的 "Service is running" 是綠燈。
    

### 3. 防火牆設定 (UFW)

雖然 RustDesk 可以打洞，但在 Linux Mint 上建議還是放行相關 Port 以確保連線穩定。

Bash

```
sudo ufw allow 21115:21119/tcp
sudo ufw allow 21116/udp
sudo ufw reload
```

---

## Part 2: NoMachine —— 區網內的效能王者

如果你是在家裡用筆電連 GPD MicroPC (或是透過 VPN/Tailscale 連線)，NoMachine 是絕對的首選。它使用獨家的 NX 協定，**剪貼簿同步、檔案拖拉、甚至看影片**都非常順暢。

### 1. 下載與安裝

NoMachine 官方網站提供 Linux `.deb` 包。

**Terminal 安裝流：**



```Bash
# 1. 下載 (建議去官網複製最新連結，這邊是範例)
wget https://download.nomachine.com/download/8.10/Linux/nomachine_8.10.1_1_amd64.deb

# 2. 安裝
sudo dpkg -i nomachine_8.10.1_1_amd64.deb
```

### 2. 設定與檢查

NoMachine 安裝完會自動啟動服務 (`/usr/NX/bin/nxserver`)。它會直接抓取你 Linux 系統的使用者帳號與密碼。

- **登入帳號**：你的 Linux 帳號 (如 `kirin`)
    
- **登入密碼**：你的 Linux 登入密碼
    
- **Port**：預設 **4000** (NX 協定)
    

### 3. 防火牆設定 (關鍵！)

NoMachine 不會自動打洞，你**必須**手動開啟防火牆 Port 4000，否則別台電腦連不進來。



```Bash
# 放行 NX 協定 Port
sudo ufw allow 4000/tcp
sudo ufw allow 4000/udp
sudo ufw reload
```


## 總結：RD 該選哪一個？

|**比較項目**|**RustDesk**|**NoMachine**|
|---|---|---|
|**最佳場景**|**外網連線 (WAN)**<br><br>  <br><br>像 TeamViewer 一樣方便，無須 VPN。|**內網連線 (LAN/VPN)**<br><br>  <br><br>追求極致畫質與低延遲。|
|**設定難度**|⭐ (極低)|⭐⭐ (需開防火牆)|
|**連線速度**|良好 (依賴中繼伺服器)|**極佳 (P2P 直連)**|
|**開源與否**|Open Source|閉源 (個人免費)|
|**多螢幕支援**|支援|支援 (且切換更順暢)|

**我的建議：**

**兩個都裝！**

- 平常在家開發回測，用 **NoMachine** 享受絲滑般的體驗。
    
- 出門在外如果 VPN 連不上，用 **RustDesk** 當作備援後門。
    

---

_Tags: #LinuxMint #RustDesk #NoMachine #RemoteDesktop #DevOps #GPDMicroPC_