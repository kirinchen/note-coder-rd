---
title: install Docker Compose And Git
tags: [linux, devops]

---

# To install Docker Compose And Git on Ubuntu 24.04:

---

### Step 1: Update System
Ensure your system packages are up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

---

To install `docker-compose` and `git` on Ubuntu 24 LTS, follow these steps:

---

### Step 2. **Install Docker **
Docker Compose can be installed using the Docker CLI plugin or as a standalone binary.


1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install Docker if not already installed:
   ```bash
   sudo apt install -y docker.io
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

5. **Verify Docker Installation:**
   ```bash
   docker --version
   ```

---

### Step 3: Install Docker Compose
1. **Download Docker Compose Binary:**
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   ```

2. **Set Permissions:**
   ```bash
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Verify Docker Compose Installation:**
   ```bash
   docker-compose --version
   ```

---

### Step 4: Add User to Docker Group (Optional)
To avoid using `sudo` for Docker commands:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

### Step 5: Enable and Start Docker Service
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

---

### Step 6: **Install Git**
1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install Git:
   ```bash
   sudo apt install -y git
   ```

3. Verify installation:
   ```bash
   git --version
   ```

---
