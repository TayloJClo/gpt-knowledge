# Taylor’s Pi System Profile

## Hardware / OS
* Raspberry Pi (running Raspberry Pi OS Lite)
* User: taylo
* Network: 192.168.178.69
* Connected over Wi-Fi

## VeraCrypt
* Container file: /mnt/external/My files.hc
* Mountpoint: /mnt/secure
* Mounted via /usr/bin/veracrypt

## Syncthing
* Web GUI: https://127.0.0.1:8384
* Folder ID: veracrypt-backup
* Folder path: /mnt/external
* Sync interval: 0 (manual scan via API)
* API key: \[REDACTED – stored on Pi]

## Docker Compose
* Compose file: /home/taylo/docker/docker-compose.yml
* Services:
  * filebrowser (mounts /mnt/secure:/srv)
  * resilio (mounts /mnt/secure:/sync)
  * pihole (DNS)
  * portainer (Docker management UI on 9443/9000)
  * **homeassistant** (host network; config at `/mnt/external/homeassistant`)
  * **n8n** (workflow automation) — image `n8nio/n8n:latest`, ports `5678:5678`, volume `/home/taylo/docker/n8n:/home/node/.n8n`, env: `GENERIC_TIMEZONE=Europe/Berlin`, `N8N_HOST=taylorspi.tailea056c.ts.net`, `WEBHOOK_URL=https://taylorspi.tailea056c.ts.net`, `N8N_SECURE_COOKIE=true`, `N8N_PROXY_HOPS=1`, `N8N_PROTOCOL=https` (plus Basic Auth + Encryption key)
  * **uptime-kuma** (self-hosted uptime monitoring)
    - Image: `louislam/uptime-kuma:1`
    - Ports: `3001:3001`
    - Volume: `/home/taylo/docker/uptime-kuma:/app/data`
    - Purpose: Internal monitoring of Home Assistant, n8n, Syncthing, FileBrowser, and system reachability
    - Access:
      - LAN: http://192.168.178.69:3001
      - Tailscale: http://taylorspi.tailea056c.ts.net:3001
    - Update: `docker compose pull uptime-kuma && docker compose up -d uptime-kuma`


* Restart policy: unless-stopped

## Backup script
* Path: /home/taylo/veracrypt_backup.sh
* Schedule: Sunday 02:00 via cron
* Steps:
  1. Stops filebrowser/resilio
  2. Dismounts VeraCrypt
  3. Triggers Syncthing rescan
  4. Waits until sync idle
  5. Remounts VeraCrypt
  6. Starts filebrowser/resilio again
## Boot Restore Script
* Path: `/usr/local/bin/restore_secure.sh`
* Purpose: Automatically mounts VeraCrypt container and restarts dependent Docker services (`filebrowser`, `resilio`) after boot.
* Runs via systemd unit: `/etc/systemd/system/restore-secure.service`
* Trigger: `multi-user.target`
* Dependencies:
  * `/mnt/external` mounted
  * `/mnt/external/My files.hc` present
  * `docker.service` active
* Behavior:
  1. Confirms `/mnt/external` mounted
  2. Mounts `/mnt/external/My files.hc` → `/mnt/secure`
  3. Starts Docker services (`filebrowser`, `resilio`)
  4. Verifies containers running

* Note: Home Assistant runs from `/mnt/external` and is **not** part of the VeraCrypt stop/start set.

## DNS / Network
* PC DNS: Pi-hole (192.168.178.69) primary, 1.1.1.1 secondary
* Pi-hole container may go down during backup

## Web UIs
- Portainer: https://192.168.178.69:9443  (or via Tailscale DNS)
- **Home Assistant:** http://192.168.178.69:8123  (or via Tailscale: `http://taylorspi.tailea056c.ts.net:8123`)
- **n8n:** https://taylorspi.tailea056c.ts.net/  *(served via Tailscale HTTPS — do **not** use :5678 in the URL)*

