# 🐳 Docker & Container Configuration — TaylorsPi

## 🧩 Overview
- **Docker Engine:** Installed and managed by systemd  
- **Compose File:** `/home/taylo/docker/docker-compose.yml`  
- **Compose Version:** v3 syntax  
- **User:** `taylo`  
- **Base OS:** Debian GNU/Linux 13 (Trixie) on ARM64  
- **Container Restart Policy:** `unless-stopped`  
- **Docker Storage Driver:** overlay2  
- **Network Mode:** Mixed (bridge + host)

---

## 📦 Active Services (from docker-compose.yml)

### 🗂️ 1️⃣ FileBrowser
A web-based file manager to browse mounted VeraCrypt storage.

| Property | Value |
|-----------|--------|
| **Image** | `filebrowser/filebrowser:latest` |
| **Container Name** | `filebrowser` |
| **Ports** | `8080:80` (HTTP web UI) |
| **Volumes** | `/mnt/secure:/srv`; `./filebrowser/database:/database`; `/mnt/storage:/srv/storage` |
| **Command** | `["--database", "/database/filebrowser.db", "--root", "/srv"]` |
| **Restart Policy** | `unless-stopped` |
| **Depends On** | VeraCrypt mount at `/mnt/secure` |

---

### 🔄 2️⃣ Resilio Sync
Peer-to-peer sync service (used before Syncthing testing).

| Property | Value |
|-----------|--------|
| **Image** | `linuxserver/resilio-sync:latest` |
| **Container Name** | `resilio` |
| **Ports** | `8888:8888` (Web UI), `55555:55555` (Sync traffic) |
| **Volumes** | `~/docker/resilio/config:/config`; `/mnt/secure:/sync` |
| **Environment Variables** | `PUID=1000`, `PGID=1000`, `TZ=Europe/Berlin` |
| **Restart Policy** | `unless-stopped` |

---

### 🧠 3️⃣ Pi-hole
DNS sinkhole and ad blocker for local network.

| Property | Value |
|-----------|--------|
| **Image** | `pihole/pihole:latest` |
| **Container Name** | `pihole` |
| **Network Mode** | `host` |
| **Environment Variables** | `TZ=Europe/Berlin`; `DNSMASQ_LISTENING=all`; `FTLCONF_LOCAL_IPV4=192.168.178.69` |
| **Volumes** | `./pihole/etc-pihole:/etc/pihole`; `./pihole/etc-dnsmasq.d:/etc/dnsmasq.d` |
| **Ports (logical)** | DNS: 53/udp, Web UI: 80/tcp (via host mode) |
| **Restart Policy** | `unless-stopped` |
| **Notes** | Runs as root in host network; used by all LAN devices. |

---

### 🧰 4️⃣ Portainer
Web UI to manage Docker containers, images, volumes, networks.

| Property | Value |
|-----------|-------|
| **Image** | portainer/portainer-ce:latest |
| **Container Name** | portainer |
| **Ports** | 9000:9000 (HTTP), 9443:9443 (HTTPS) |
| **Volumes** | /var/run/docker.sock:/var/run/docker.sock ; ./portainer/data:/data |
| **Restart Policy** | unless-stopped |
| **Notes** | Local admin UI; reachable via LAN or Tailscale HTTPS. |

---

### 🏠 5️⃣ Home Assistant  *(NEW)*
| Property | Value |
|-----------|--------|
| **Image** | ghcr.io/home-assistant/home-assistant:stable |
| **Container Name** | homeassistant |
| **Ports** | 8123 (via host networking) |
| **Volumes** | `/mnt/external/homeassistant:/config` |
| **Network Mode** | host |
| **Restart Policy** | unless-stopped |
| **Purpose** | Smart home automation platform, accessible over LAN or Tailscale |
| **Notes** | Runs from unencrypted SSD path; not tied to VeraCrypt lifecycle. |

---

### ⚙️ 6️⃣ n8n (Workflow Automation) — *NEW*
| Property | Value |
|-----------|------|
| **Image** | `n8nio/n8n:latest` |
| **Container Name** | `n8n` |
| **Ports** | `5678:5678` (internal HTTP; not used directly when served via Tailscale) |
| **Volumes** | `/home/taylo/docker/n8n:/home/node/.n8n` |
| **Environment** | `GENERIC_TIMEZONE=Europe/Berlin`, `N8N_HOST=taylorspi.tailea056c.ts.net`, `WEBHOOK_URL=https://taylorspi.tailea056c.ts.net`, `N8N_SECURE_COOKIE=true`, `N8N_PROXY_HOPS=1`, `N8N_PROTOCOL=https`, `N8N_BASIC_AUTH_*` (set), `N8N_ENCRYPTION_KEY` (set) |
| **Restart Policy** | `unless-stopped` |
| **Access** | **HTTPS (Tailscale Serve)** → `https://taylorspi.tailea056c.ts.net/` |
| **Notes** | Served via Tailscale HTTPS at the root path (`/`). Do *not* browse to `:5678` directly. Use the tailnet URL without a port. |

### 🧭 7️⃣ Uptime Kuma (Self-Hosted Monitor)
| Property | Value |
|-----------|-------|
| **Image** | `louislam/uptime-kuma:1` |
| **Container Name** | `uptime-kuma` |
| **Ports** | `3001:3001` |
| **Volumes** | `/home/taylo/docker/uptime-kuma:/app/data` |
| **Network Mode** | bridge |
| **Restart Policy** | unless-stopped |
| **Purpose** | Local uptime monitoring for Home Assistant, n8n, FileBrowser, Syncthing, Pi-hole |
| **Update Command** | `docker compose pull uptime-kuma && docker compose up -d uptime-kuma` |
| **Access URLs** | LAN: `http://192.168.178.69:3001`, Tailscale: `http://taylorspi.tailea056c.ts.net:3001` |


#### Serve mapping (Tailscale)

tailscale serve status
https://taylorspi.tailea056c.ts.net
 (tailnet only)
|-- / proxy http://127.0.0.1:5678

## 🧱 Volumes and Mounts Summary
| Host Path | Container Path | Used By | Description |
|-----------|----------------|---------|-------------|
| `/mnt/secure` | `/srv` / `/sync` | FileBrowser / Resilio | Decrypted VeraCrypt volume |
| `/mnt/storage` | `/srv/storage` | FileBrowser | General storage area |
| `~/docker/resilio/config` | `/config` | Resilio | Persistent settings |
| `./filebrowser/database` | `/database` | FileBrowser | User DB and config |
| `./pihole/etc-pihole` | `/etc/pihole` | Pi-hole | DNS data and logs |
| `./pihole/etc-dnsmasq.d` | `/etc/dnsmasq.d` | Pi-hole | DHCP/dnsmasq configs |
| **`/mnt/external/homeassistant`** | **`/config`** | **Home Assistant** | **SSD-based HA configuration** |
**Note:**  
`filebrowser` and `resilio` now auto-start only after `/mnt/secure` is verified by the boot-restore script.  
This prevents failed mounts or corruption if the encrypted container isn’t ready.

---

## 🔌 Networking Summary
| Network | Type | Used By | Notes |
|--------|------|---------|------|
| **bridge** | Default Docker network | FileBrowser, Resilio | Standard isolated container network |
| **host** | Direct Pi network | Pi-hole, **Home Assistant** | Needed for DNS and HA discovery |
| **localhost (127.0.0.1)** | Bound services | Syncthing REST API & GUI | Not accessible remotely |
