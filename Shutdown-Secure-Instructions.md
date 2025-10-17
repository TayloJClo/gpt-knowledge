# 🔒 Raspberry Pi — Secure Shutdown Procedure

This file documents how to safely shut down *Taylor’s Pi* while protecting the VeraCrypt container and Docker services.

---

## 🧩 Overview
System references:
- User: `taylo`
- VeraCrypt container: `/mnt/external/My files.hc`
- Mount point: `/mnt/secure`
- Docker Compose directory: `/home/taylo/docker`
- Key services using VeraCrypt: `filebrowser`, `resilio`
- Script location (new): `/usr/local/bin/shutdown_secure.sh`

---

## ⚙️ Script — `/usr/local/bin/shutdown_secure.sh`
```bash
#!/bin/bash
set -euo pipefail

# --- Paths from your system profile ---
VERACRYPT=/usr/bin/veracrypt
DOCKER=/usr/bin/docker
COMPOSE_DIR="/home/taylo/docker"
MOUNTPOINT="/mnt/secure"

echo "[+] Stopping Docker services that use VeraCrypt ..."
cd "$COMPOSE_DIR"
$DOCKER compose stop filebrowser resilio || true

echo "[+] Dismounting VeraCrypt container ..."
if mountpoint -q "$MOUNTPOINT"; then
    $VERACRYPT --dismount "$MOUNTPOINT" || {
        echo "[-] Normal dismount failed, forcing..."
        $VERACRYPT --force --dismount "$MOUNTPOINT"
    }
else
    echo "[=] VeraCrypt volume already dismounted."
fi

echo "[+] Waiting for filesystem flush ..."
sleep 5

echo "[+] Shutting down the system safely ..."
/usr/sbin/shutdown -h now

🛠️ Installation Steps

sudo nano /usr/local/bin/shutdown_secure.sh
# Paste script contents

sudo chmod +x /usr/local/bin/shutdown_secure.sh

## 🔄 Auto-Restore Behavior
After safe shutdown and power-up:
- The systemd service `restore-secure.service` triggers `/usr/local/bin/restore_secure.sh`.
- This script checks `/mnt/external`, mounts the VeraCrypt container to `/mnt/secure`, and starts the required Docker services.
- No manual steps are required post-boot (password prompt appears only if volume is unmounted).

---

  # NEW: Home Assistant on external SSD (unencrypted)
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Berlin
    volumes:
      - /mnt/external/homeassistant:/config


