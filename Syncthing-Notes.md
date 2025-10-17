# Syncthing Configuration (Taylor’s Pi)

## Device Info
- **Device name:** TaylorsPi
- **Runs as user:** taylo
- **Service file:** /usr/lib/systemd/system/syncthing@.service
- **Config directory:** /home/taylo/.local/state/syncthing/
- **Web GUI address:** https://127.0.0.1:8384
- **API key:** [REDACTED]
- **Access:** Localhost only (bound to 127.0.0.1)
- **TLS:** Enabled (HTTPS)

## Network and Discovery
- **Sync port:** 22000/TCP  
- **Local discovery port:** 21027/UDP  
- **Global discovery:** Enabled  
- **NAT/Relay:** Auto (UPnP attempts visible in logs)  
- **Web GUI port:** 8384 (localhost only)

## Folders
### 1️⃣ `veracrypt-backup`
- **Folder ID:** `veracrypt-backup`
- **Path on Pi:** `/mnt/external`
- **Purpose:** Sync VeraCrypt container file (`My files.hc`) to main PC.
- **Type:** Send (send)
- **Rescan interval:** `0` (manual rescan only)
- **Versioning:** Disabled
- **Ignore patterns:**  
  ```text
  !My files.hc
  *

---

# Network-Info.md  
```markdown
# 🌐 Network Configuration — TaylorsPi

## 📍 Basic Network Details
| Item | Value |
|------|------|
| **Hostname** | TaylorsPi |
| **Primary Connection** | Wi-Fi |
| **Local IP Address** | 192.168.178.69 |
| **Default Gateway** | 192.168.178.1 |
| **Network Mode** | DHCP (via Fritz!Box) |
| **DNS Provider** | Pi-hole (local) + Cloudflare fallback |
| **Time Zone** | Europe/Berlin |

---

## 🧩 DNS Configuration
**Primary DNS:** `192.168.178.69` → Pi-hole container  
**Secondary DNS:** `1.1.1.1` (Cloudflare)

Windows systems are configured manually:
```powershell
Set-DnsClientServerAddress -InterfaceAlias "Wi-Fi 3" -ServerAddresses ("192.168.178.69","1.1.1.1")

🧾 Notes

The Pi uses Pi-hole for all DNS queries; the PC also points to the Pi-hole for DNS.

Pi-hole runs in host networking mode, sharing the same IP as the Pi itself.

Syncthing uses direct device connections when available; falls back to relays if NAT traversal fails.

VeraCrypt and FileBrowser share the /mnt/secure mount; only accessible locally or through authenticated web UIs.

Home Assistant (added) is reachable on the LAN and via Tailscale:

LAN: http://192.168.178.69:8123

Tailscale: http://taylorspi.tailea056c.ts.net:8123

✅ Summary
TaylorsPi acts as both a DNS server (Pi-hole) and sync node (Syncthing + Resilio) within a home LAN on subnet 192.168.178.0/24.
It connects via Wi-Fi to a Fritz!Box router, exposes minimal external ports, and isolates all sensitive interfaces (like Syncthing GUI) to localhost for security.
It now also runs Home Assistant for local smart-home automation.


---

# veracrypt_backup.sh  
```bash
#!/bin/bash
set -euo pipefail

# --- full paths (cron-safe) ---
VERACRYPT=/usr/bin/veracrypt
CURL=/usr/bin/curl
DOCKER=/usr/bin/docker
DATE=/usr/bin/date
SLEEP=/usr/bin/sleep
AWK=/usr/bin/awk
MOUNTPOINT_BIN=/usr/bin/mountpoint

# --- your settings (do not change unless you want to) ---
CONTAINER="/mnt/external/My files.hc"   # VeraCrypt container file
MOUNTPOINT="/mnt/secure"                # mountpoint for container
FOLDER_ID="veracrypt-backup"            # Syncthing Folder ID
VC_PASSWORD="Tj*******71!"              # stored in script (you chose this)

# --- docker-compose settings ---
COMPOSE_DIR="/home/taylo/docker"        # folder containing docker-compose.yml
COMPOSE_FILE="docker-compose.yml"       # compose filename
FB_SERVICES="filebrowser resilio"       # compose service names to stop/start

# --- Syncthing REST settings (local HTTPS allowed) ---
CONFIG="/home/taylo/.local/state/syncthing/config.xml"
API_KEY="$(grep -oP '(?<=<apikey>)[^<]+' "$CONFIG")"
TLS="$(grep -oP '(?<=<gui[^>]*tls=")[^"]+' "$CONFIG" | tr 'A-Z' 'a-z' || true)"
API_BASE="http://127.0.0.1:8384"
CURL_BIN="/usr/bin/curl -fsS"
if [ "$TLS" = "true" ]; then
  API_BASE="https://127.0.0.1:8384"
  CURL_BIN="/usr/bin/curl -kfsS"
fi
AUTH_HDR="X-API-Key: ${API_KEY}"

log(){ echo "[$($DATE +'%F %T')] $*"; }

compose(){
  ( cd "$COMPOSE_DIR" && $DOCKER compose -f "$COMPOSE_FILE" "$@" )
}

stop_compose_services(){
  if [ -f "$COMPOSE_DIR/$COMPOSE_FILE" ]; then
    log "Stopping compose services: $FB_SERVICES (in $COMPOSE_DIR)"
    ( cd "$COMPOSE_DIR" && $DOCKER compose -f "$COMPOSE_FILE" stop $FB_SERVICES ) || true
  else
    log "Compose file not found at $COMPOSE_DIR/$COMPOSE_FILE — skipping stop."
  fi
}

start_compose_services(){
  if [ -f "$COMPOSE_DIR/$COMPOSE_FILE" ]; then
    log "Starting compose services: $FB_SERVICES (in $COMPOSE_DIR)"
    ( cd "$COMPOSE_DIR" && $DOCKER compose -f "$COMPOSE_FILE" start $FB_SERVICES ) || true
  else
    log "Compose file not found at $COMPOSE_DIR/$COMPOSE_FILE — skipping start."
  fi
}

log "Weekly VeraCrypt backup: start"

# ensure not in mount dir
cd ~

# stop compose services to free mount
stop_compose_services

# 1) Unmount if mounted
if $MOUNTPOINT_BIN -q "$MOUNTPOINT"; then
  log "Dismounting $MOUNTPOINT ..."
  $VERACRYPT --dismount "$MOUNTPOINT" || {
    log "Normal dismount failed — forcing ..."
    $VERACRYPT --force --dismount "$MOUNTPOINT" || true
  }
  $SLEEP 10
else
  log "Mountpoint not active; nothing to dismount."
fi

# 2) Trigger Syncthing scan via REST
log "Triggering Syncthing scan for folder '$FOLDER_ID' ..."
$CURL_BIN -X POST -H "$AUTH_HDR" "${API_BASE}/rest/db/scan?folder=${FOLDER_ID}" >/dev/null || true

# 3) Wait until Syncthing finishes syncing (idle + needBytes=0)
log "Waiting for Syncthing to finish syncing ..."
ATTEMPTS=0
MAX_ATTEMPTS=720  # ~2 hours at 10s intervals

while true; do
  STATUS_JSON="$($CURL_BIN -H "$AUTH_HDR" "${API_BASE}/rest/db/status?folder=${FOLDER_ID}" || true)"
  STATE="$(echo "$STATUS_JSON" | $AWK -F'"' '/"state":/ {print $4; exit}')"
  NEED_BYTES="$(echo "$STATUS_JSON" | $AWK -F'[:,}]' '/"needBytes":/ {print $3; exit}')"
  [ -z "${STATE:-}" ] && STATE="unknown"
  [ -z "${NEED_BYTES:-}" ] && NEED_BYTES=0

  log "State=$STATE needBytes=${NEED_BYTES}"

  if [[ "$STATE" == "idle" && "$NEED_BYTES" -eq 0 ]]; then
    log "Sync complete."
    break
  fi

  ATTEMPTS=$((ATTEMPTS+1))
  if (( ATTEMPTS > MAX_ATTEMPTS )); then
    log "Timeout waiting for sync; proceeding to remount."
    break
  fi

  $SLEEP 10
done

# 4) Remount the VeraCrypt container
log "Remounting VeraCrypt container ..."
$VERACRYPT --text --non-interactive --password "$VC_PASSWORD" "$CONTAINER" "$MOUNTPOINT" || {
  log "Remount failed."
}

# 5) Start compose services again
start_compose_services

log "Weekly VeraCrypt backup: done"

