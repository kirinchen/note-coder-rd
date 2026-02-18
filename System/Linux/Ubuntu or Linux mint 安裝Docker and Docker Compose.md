身為 IT RD，安裝這兩套軟體有「乾淨、正規」的最佳實踐。特別是 **Docker**，在 Linux Mint 上安裝官方源（Official Repo）比用系統內建的舊版好太多了。

以下是針對開發者環境的標準安裝流程：

---

### Part 1: 安裝 Docker (官方最新版)

**注意：** Linux Mint 的軟體庫裡有 `docker.io`，但那個版本通常比較舊。我們直接抓 Docker 官方的來裝。

**1. 移除舊版 (如果有)**

以防萬一，先清乾淨：



```Bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**2. 安裝前置依賴**



```Bash
sudo apt update
sudo apt install ca-certificates curl gnupg
```

**3. 加入 Docker 官方 GPG Key**



```Bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**4. 加入 Docker Repository (Mint 專用寫法)**

_IT RD 請注意：這裡不能直接用 `$(lsb_release -cs)`，因為 Mint 會回傳自己的代號 (例如 `wilma`)，Docker 認不得。我們要抓它底層對應的 Ubuntu 代號 (例如 `noble` 或 `jammy`)。_

請直接複製這整段執行：



```Bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$UBUNTU_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**5. 安裝 Docker Engine & Docker Compose**



```Bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**6. 設定免 Sudo 執行 (關鍵步驟)**

身為開發者，你不會想每次打 `docker ps` 都要加 `sudo`。把你的帳號加入 docker 群組：



```Bash
sudo usermod -aG docker $USER
```

**_做完這步後，請務必「登出再登入」或重開機，權限才會生效。_**

**7. 驗證**

重登後，輸入 `docker run hello-world`，如果有看到 Hello from Docker 就成功了。

---



