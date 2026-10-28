---
title: OmniArchiver-F2L
emoji: 🌐
colorFrom: blue
colorTo: indigo
sdk: docker
app_port: 7860
pinned: false
---

<div align="center"><h1>🌐 OmniArchiver-F2L</h1>
<b>A high-performance Python Telegram bot to generate permanent HTTP Stream & Direct Download links from your Telegram channels without duplicating files, powered by MongoDB and Multi-Bot parallel streaming.</b>
</div><br>

**Hugging Face Spaces:** Follow [HUGGING_FACE.md](HUGGING_FACE.md) to deploy this version. It uses a Docker Space on port **7860**, detects its public URL automatically, and reads credentials from Space secrets. Configure your external MongoDB URI as `DATABASE_URL`; see the guide for network troubleshooting if the startup database ping fails.

**Cloudflare Worker:** Follow [CLOUDFLARE.md](CLOUDFLARE.md) to deploy the streaming reverse proxy through GitHub Actions, then use its URL as the bot's `BASE_URL`.

## **📑 INDEX**

* [**✨ Features**](#features)
* [**⚙️ Installation**](#installation)
* [**📝 Variables**](#variables)
* [**🎮 Commands & Usage**](#commands)
* [**🕹 Deployment**](#deployment)
  * [Hugging Face Spaces](HUGGING_FACE.md)
  * [Linux / VPS Deployment](#d-vps)
  * [Docker Deployment](#d-docker)
  * [Local Deployment](#d-local)
* [**❤️ Credits**](#credits)

---

<a name="features"></a>

## ✨ Features

- **🚀 Multi-Bot Worker Pool:** Support for multiple bot tokens to parallelize chunk transmissions and eliminate streaming bottlenecks.
- **🗄️ Organized MongoDB Storage:** Neatly partitions data into separate collections (`movies`, `anime`, `webseries`, `direct_files`).
- **🚫 Zero File Duplication:** Directly indexes files from your existing channels without needing a storage bin channel.
- **🎬 `/link` Command:** Generate instant permanent stream & download links for single movies/files from any channel.
- **📺 `/batch` Command:** Batch index full Anime or Web Series seasons by specifying start and end message links.
- **👥 `AUTH_USERS` Permissions:** Multi-admin support allowing designated users full access to manage and index files.
- **⚡ Hardware Accelerated:** Powered by `tgcrypto` and asynchronous chunk streaming (`Quart`/`Uvicorn`).
- **📦 1-Click Database Backup:** Instant `/backup` command dumps your entire MongoDB database into a compressed archive sent directly to Telegram DM.

---

<a name="installation"></a>

## ⚙️ Installation

**1. Clone the repository:**
```bash
git clone https://github.com/WorkerOfArea51/OmniArchiver-F2L.git
cd OmniArchiver-F2L
```

**2. Create a virtual environment & install requirements:**
```bash
python3.11 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

---

<a name="variables"></a>

## 📝 Variables

Configure these variables in your `.env` file or hosting environment:

| Variable | Required | Description |
| :--- | :--- | :--- |
| `TELEGRAM_API_ID` | **Yes** | Telegram API ID from [my.telegram.org](https://my.telegram.org) (`int`) |
| `TELEGRAM_API_HASH` | **Yes** | Telegram API Hash from [my.telegram.org](https://my.telegram.org) (`str`) |
| `TELEGRAM_BOT_TOKEN` | **Yes** | Main Telegram bot token from [@BotFather](https://t.me/BotFather) (`str`) |
| `TELEGRAM_BOT_USERNAME` | *Optional* | Bot username without `@`; detected automatically on startup |
| `MULTI_BOT_TOKENS` | *Optional* | Additional worker bot tokens (space-separated) for parallel stream acceleration |
| `OWNER_ID` | **Yes** | Your numeric Telegram user ID (`int`) |
| `AUTH_USERS` | *Optional* | Space-separated list of Telegram user IDs with full admin rights |
| `ALLOWED_USER_IDS` | *Optional* | Allowed user IDs for general bot usage (leave empty to allow everyone) |
| `TELEGRAM_CHANNEL_ID` | *Optional* | Fallback Storage Channel ID (with `-100` prefix). **Only needed if uploading files directly via private DM to the bot**. If you only index existing channels using `/link` or `/batch`, leave this empty! |
| `DATABASE_URL` | **Yes** | MongoDB connection URI (e.g. `mongodb+srv://...` or `mongodb://localhost:27017`) |
| `DATABASE_NAME` | *Optional* | Database name in MongoDB (default: `OmniArchiver`) |
| `BASE_URL` | **Yes outside Spaces** | Public server URL; defaults to `https://SPACE_HOST` on Hugging Face Spaces |
| `BIND_ADDRESS` | *Optional* | Bind address (default: `0.0.0.0`) |
| `PORT` | *Optional* | Port to listen on (default: `8080`) |
| `OMNI_MONGO_URI` | *Optional* | Separate upstream OmniArchiverDB URI; otherwise uses your configured MongoDB server |

Environment variables take precedence over local configuration files. The Docker image defaults to port `7860`; local Python runs default to `8080`.

---

<a name="commands"></a>

## 🎮 Commands & Usage

### 🎬 Single Movie Indexing (`/link`)
Generate a permanent stream and download link for a movie in your channel:
```text
/link https://t.me/c/1234567890/42
```
*(Or forward a channel post into the bot and reply with `/link`)*

---

### 📺 Anime Batch Indexing (`/batch anime`)
Index an entire season of Anime episodes into the `anime` collection:
```text
/batch anime https://t.me/c/1234567890/10 https://t.me/c/1234567890/22
```

---

### 🍿 Web Series Batch Indexing (`/batch series`)
Index Web Series episodes into the `webseries` collection:
```text
/batch series https://t.me/c/1234567890/50 https://t.me/c/1234567890/60
```

---

### 📊 Other Commands
- `/stats` - View live database records, RAM, CPU, worker bots & streaming bandwidth (Admin only).
- `/backup` - Instant 1-click compressed `.json.gz` database backup sent to your Telegram DM (Admin only).
- `/clean` - Flush RAM caches & compact memory (Admin only).
- `/restart` - Smoothly restart the bot process (Admin only).
- `/sync_duration` - Auto-backfill video durations for existing database records (Admin only).
- `/revoke <code_or_url>` - Delete a file link from MongoDB so it can be re-indexed cleanly (Admin only).
- `/sh <cmd>` - Execute a shell terminal command directly from Telegram (Admin only).
- `/log` - Download bot event log file (Admin only).
- `/privacy` - View privacy policy.
- `/help` - View command guide.

---

<a name="deployment"></a>

## 🕹 Deployment

<a name="d-vps"></a>

### 🐧 Linux / Cloud VPS Deployment (Ubuntu / Debian / FreeBSD / Any VPS)

**1. Clone the repository & set up Python 3.11+ environment:**
```bash
cd ~
git clone https://github.com/WorkerOfArea51/OmniArchiver-F2L.git
cd OmniArchiver-F2L
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

**2. Configure your `.env` file:**
```bash
cp .env.example .env
nano .env
```
Fill in your Telegram API credentials, Bot Token, Owner ID, and MongoDB connection string.

**3. Run 24/7 in the Background:**

#### Option A: Systemd Service (Recommended for Ubuntu / Debian)
Create a service file:
```bash
sudo nano /etc/systemd/system/omniarchiver.service
```
Paste the configuration (replace `your_user` and path with yours):
```ini
[Unit]
Description=OmniArchiver-F2L Telegram Streaming Bot
After=network.target

[Service]
Type=simple
User=your_user
WorkingDirectory=/home/your_user/OmniArchiver-F2L
ExecStart=/home/your_user/OmniArchiver-F2L/venv/bin/python -m bot
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable omniarchiver
sudo systemctl start omniarchiver
```

#### Option B: Background Run + Self-Healing Crontab Watchdog
If your hosting does not support `systemd` (e.g. FreeBSD, shared shells, or non-root VPS):
```bash
# Start in background
nohup ./venv/bin/python -m bot > bot.log 2>&1 &
```
To ensure it automatically turns back on if the server restarts or reboots, add the included watchdog to your crontab:
```bash
chmod +x scripts/watchdog.sh
crontab -e
```
Add this line:
```cron
*/5 * * * * /path/to/OmniArchiver-F2L/scripts/watchdog.sh
```

---

<a name="d-docker"></a>

### 🐳 Docker Deployment
```bash
docker build -t omniarchiver-f2l .
docker run -d --name omniarchiver -p 8080:7860 -e PORT=7860 --env-file .env --restart unless-stopped omniarchiver-f2l
```

---

<a name="d-local"></a>

### 💻 Local Development
```bash
source venv/bin/activate  # On Windows: venv\Scripts\activate
python -m bot
```

---

<a name="credits"></a>

## ❤️ Credits

- [**WorkerOfArea51**](https://github.com/WorkerOfArea51): Maintainer of OmniArchiver-F2L.
