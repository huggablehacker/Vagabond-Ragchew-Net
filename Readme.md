# Vagabond Ragchew Net
*Moving In the Right Direction*

A self-hosted web application for the [Vagabond Ragchew Net](http://vagabondnet.net) — a daily amateur radio net based in Winston-Salem, NC, operating since 1991.

---

## What it does

- **Public website** — club history, check-in procedure, membership info, pledge, newsletter archive, and club bylaws — all served locally with no external dependencies
- **FCC Callsign Lookup** — searches 750,000+ amateur radio licenses from a local SQLite copy of the FCC ULS database, with QSL card composer and contact logging
- **Automatic database updates** — the FCC license database rebuilds every 24 hours via a background scheduler

---

## Stack

| Layer | Tech |
|---|---|
| Web server | Flask (Python) |
| Database | SQLite |
| Proxy | nginx |
| Scheduler | Python subprocess watchdog |
| Frontend | Single-file HTML/CSS/JS — no framework, no build step |

---

## Project structure

```
VBN/
├── vagabondnet.html       ← full single-page website
├── app.py                 ← Flask server + API
├── scheduler.py           ← watchdog + 24h DB rebuilder
├── import_data.py         ← FCC ULS data downloader/importer
├── requirements.txt       ← Python dependencies (just Flask)
├── vagabond_nginx.conf    ← nginx reverse proxy config
├── start.sh               ← Linux/macOS launcher
├── start.bat              ← Windows launcher
├── .env.example           ← environment variable template
├── converter/
│   └── netlog.py          ← NetLogger .log file parser
├── files/                 ← PDFs served at /files/<name>
│   ├── bylaws.pdf
│   ├── NetControlScript.pdf
│   └── *.pdf              ← newsletter archive

```

Files created at runtime (excluded from git):
```
├── .venv/                 ← Python virtual environment
├── ham_radio.db           ← FCC license database (~400 MB)
├── checkins.db            ← archived net check-in sessions
├── scheduler.log          ← combined app + scheduler log
├── scheduler_status.json  ← live rebuild countdown
└── l_amat.zip             ← FCC data download (deleted after import)
```

---

## Quick start

### Requirements
- Python 3.9+
- nginx
- Linux, macOS, or Windows

### 1. Clone and configure

```bash
git clone https://github.com/yourusername/vagabond-net.git
cd vagabond-net

# Set your admin password
cp .env.example .env
nano .env   # change ADMIN_PASSWORD
```

### 2. Install nginx and configure the proxy

```bash
sudo apt install nginx   # Debian/Ubuntu

sudo cp vagabond_nginx.conf /etc/nginx/sites-available/vagabond
sudo ln -s /etc/nginx/sites-available/vagabond /etc/nginx/sites-enabled/vagabond
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl enable nginx && sudo systemctl start nginx
```

### 3. First run — downloads FCC database (~150 MB, ~2 min)

```bash
chmod +x start.sh
./start.sh
```

### 4. Open the site

```
http://localhost
```


## Configuration


## Running as a systemd service (auto-start on boot)

```bash
sudo nano /etc/systemd/system/vagabond.service
```

```ini
[Unit]
Description=Vagabond Ragchew Net Server
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/VBN
ExecStart=/root/VBN/.venv/bin/python /root/VBN/scheduler.py
Restart=on-failure
RestartSec=10
StandardOutput=append:/root/VBN/scheduler.log
StandardError=append:/root/VBN/scheduler.log

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable vagabond
sudo systemctl start vagabond
```

---


## Adding newsletter issues

1. Drop the PDF in `files/` (e.g. `files/june26.pdf`)
2. Edit `vagabondnet.html` — find the `<!-- NEWSLETTER ARCHIVE -->` section
3. Update the featured card and add a grid entry for the previous issue
4. No restart needed — Flask serves files live

---

## FCC database

The callsign lookup uses a local SQLite copy of the FCC ULS amateur radio license database (~750,000 operators). It is downloaded and imported on first run, then rebuilt automatically every 24 hours by the scheduler.

To force an immediate rebuild:
```bash
sudo systemctl stop vagabond
.venv/bin/python import_data.py
sudo systemctl start vagabond
```

---

## Troubleshooting

**Port 80 in use by old process**
```bash
sudo ss -tlnp | grep :80
sudo kill <PID>
sudo systemctl start nginx
```

**Database not found**
```bash
.venv/bin/python import_data.py
```

**Check logs**
```bash
tail -f /root/VBN/scheduler.log
sudo journalctl -u vagabond -f
```

---

## License

MIT — see [LICENSE](LICENSE)

---

## Contact

Vagabond Ragchew Net · Winston-Salem, NC  
vagabondragchewnet@gmail.com

*73 de all the Vagabonds!*
