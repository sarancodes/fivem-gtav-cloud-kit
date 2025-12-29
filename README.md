# FiveM GTA V Cloud Kit – Host Your Own FiveM Server on AWS / VPS

![FiveM](https://img.shields.io/badge/FiveM-Server-orange?style=for-the-badge&logo=fivem)
![Linux](https://img.shields.io/badge/Linux-Ubuntu%2FDebian-blue?style=for-the-badge&logo=linux)
![MariaDB](https://img.shields.io/badge/Database-MariaDB-white?style=for-the-badge&logo=mariadb)
![txAdmin](https://img.shields.io/badge/Admin-txAdmin-green?style=for-the-badge)

A complete step-by-step guide to hosting a FiveM server on a Linux VPS. This guide covers system dependencies, firewall rules, database setup, and configuring the server via the **txAdmin Web Interface**.

## 💻 Minimum Cloud System Requirements

- **CPU:** 2 vCPU (AWS t3.small or equivalent)
- **RAM:** 2 GB (4 GB recommended)
- **Storage:** 16 GB SSD (25 GB recommended)
- **OS:** Ubuntu 20.04 / 22.04
- **Network:** Public IPv4 with ports 30120 TCP & UDP open

> Suitable for small FiveM servers (1–3 players).


---

## 🛠️ 1. Prerequisites & Dependencies

First, update your system and install the required tools.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl wget screen xz-utils
```

---

## 🛡️ 2. Firewall Rules & Ports

You must open specific ports for players to connect and for you to access the admin dashboard.

### Cloud Provider Firewall (AWS/Azure/GCP)
If you are on AWS EC2, Google Cloud, or Azure, you must also add these rules in your **Security Group (Inbound rules)** or **VPC Firewall**:

| Type | Protocol | Port Range | Source | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Custom TCP** | TCP | `30120` | `0.0.0.0/0` | FiveM Game Connection |
| **Custom UDP** | UDP | `30120` | `0.0.0.0/0` | FiveM Game Transport |
| **Custom TCP** | TCP | `40120` | `0.0.0.0/0` | txAdmin Web Panel |

---

## 🗄️ 3. Database Setup (MariaDB)

Install the database server and create a user. You will need these credentials later in the txAdmin setup wizard.

### Install & Secure
```bash
# Install MariaDB
sudo apt install mariadb-server -y

# Secure installation (Answer 'Y' to options, set a root password)
sudo mysql_secure_installation
```

### Create Database User
Enter the MySQL shell:
```bash
sudo mariadb
```

Paste the following SQL commands (change `your_secure_password`!):
```sql
CREATE USER 'qbox'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON *.* TO 'qbox'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
exit
```

---

## ⚙️ 4. Install Server Artifacts

Set up the folder structure and download the FXServer artifacts.

```bash
# Create directories
mkdir -p ~/fivem/server
mkdir -p ~/fivem/server-data
```
```bash
# Download & Extract Artifacts
cd ~/fivem/server
```
```bash
wget https://runtime.fivem.net/artifacts/fivem/build_proot_linux/master/23683-1062db8a7b8e0c03f7c159be4cbfa181f49b2cc1/fx.tar.xz
```
```bash
tar -xf fx.tar.xz
rm fx.tar.xz
```
```

---

## 🚀 5. Start Server & Get txAdmin PIN

We start the server now to generate the txAdmin PIN code.

```bash
# Start a new screen session
screen -S fivem

# Run the server script
cd ~/fivem/server
bash run.sh
```

**⚠️ Important:**
Look at the terminal output. You will see a message like:
> `txAdmin is hosted at http://0.0.0.0:40120`
> `PIN: 1234`

**Note down this PIN code.**

*(To detach from the screen without stopping the server, press `CTRL+A`, then `D`)*.

---

## 🌐 6. txAdmin Web Configuration

Now, configure the server using your web browser.

1.  **Open Browser:** Go to `http://YOUR_SERVER_IP:40120`
2.  **Link Account:** Enter the **PIN** from the terminal and link your CFX.re account.
3.  **Deployment Setup:**
    * Click **"Next"** through the welcome steps.
    * **Server Name:** Enter your desired server name.
    * **Deployment Type:** Select **"Popular Template"** (recommended for beginners) or "Custom".
    * **Template:** Choose **Qbox**, **ESX**, or **QBCore**.
    * **Data Location:** It should auto-detect or ask for a path. Use: `~/fivem/server-data`.
4.  **Final Configuration:**
    * **License Key:** Paste your key from [FiveM Keymaster](https://keymaster.fivem.net/).
    * **Database Config:** Enter the credentials you created in Step 3:
        * **Host:** `127.0.0.1` or `localhost`
        * **User:** `qbox`
        * **Password:** `your_secure_password`
        * **Database:** (Enter a name, e.g., `qbox_fw`) - *txAdmin will create this for you.*
5.  **Run Recipe:** Click "Run Recipe" to install all resources and finish setup.

---

## 🎮 7. How to Connect

Once txAdmin says the server is "Live", you can join.

### Option 1: Direct IP Connect
1. Open FiveM.
2. Press `F8` to open the console.
3. Type:
```bash
connect <SERVER_PUBLIC_IP>:30120
# Example: connect 13.233.xx.xx:30120
```

### Option 2: cfx.re Join URL
If you have a cfx.re URL (found in txAdmin dashboard):
```text
[https://cfx.re/join/abcd12](https://cfx.re/join/abcd12)
```

---

## 🔧 Troubleshooting

**Zombie Process / Port Locked**
If the server crashes but the port remains locked:

```bash
# Find the Process ID (PID)
sudo lsof -i :30120

# Kill the process (Replace 1234 with the actual PID)
sudo kill -9 1234
```
