# 🐳 Docker & Container Configuration — TaylorsPi

(Updated 2025-10-17 20:33 CEST)

# Docker Environment — TaylorsPi

## 🧩 General Information
| Item | Value |
|------|--------|
| **Docker Engine Version** | 28.5.1 |
| **Storage Driver** | overlay2 |
| **Compose File Path** | `/home/taylo/docker/docker-compose.yml` |
| **Compose File SHA256** | `8d5d02f7aeb8926039d8ed2ba01fb7c1b0bbdae6431adacba853590f3038ee16` |
| **Owner / Group** | taylo:taylo |
| **Restart Policy** | unless-stopped |
| **Docker Root Directory** | `/var/lib/docker` |

---

## 🗂️ Active Containers
| Container | Image | Ports | Status | Notes |
|------------|--------|--------|---------|-------|
| **filebrowser** | `filebrowser/filebrowser:latest` | 8080→80 | Up (healthy) | Access to decrypted `/mnt/secure` |
| **resilio** | `linuxserver/resilio-sync` | 8888, 55555 | Up | Syncs secure data folder |
| **pihole** | `pihole/pihole:latest` | 53, 80 | Up (healthy) | DNS server for LAN |
| **portainer** | `portainer/portainer-ce:latest` | 9443, 9000 | Up | Docker management web UI |
| **homeassistant** | `ghcr.io/home-assistant/home-assistant:stable` | 8123 | Up | Local smart-home hub |
| **n8n** | `n8nio/n8n:latest` | 5678 | Up | Workflow automation (via Tailscale HTTPS) |
| **uptime-kuma** | `louislam/uptime-kuma:1` | 3001 | Up (healthy) | Internal monitoring for Pi services |
| **glances** | `nicolargo/glances:latest` | (auto) | Up | System metrics dashboard |

---

## 📦 Docker Compose Configuration (summary)
```yaml
services:
  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    ports:
      - "8080:80"
    volumes:
      - /mnt/secure:/srv
      - ./filebrowser/database:/database
      - /mnt/storage:/srv/storage:rw
    restart: unless-stopped

  resilio:
    image: linuxserver/resilio-sync
    container_name: resilio
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - /home/taylo/docker/resilio/config:/config
      - /mnt/secure:/sync
    ports:
      - 8888:8888
      - 55555:55555
    restart: unless-stopped

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    network_mode: host
    restart: unless-stopped

⚙️ Service Access Summary
Service	LAN URL	Tailscale URL
FileBrowser	http://192.168.178.69:8080
	(local only)
Pi-hole	http://192.168.178.69/admin
	(host mode)
Portainer	https://192.168.178.69:9443
	https://taylorspi.tailea056c.ts.net:9443

Home Assistant	http://192.168.178.69:8123
	http://taylorspi.tailea056c.ts.net:8123

n8n	http://192.168.178.69:5678
	https://taylorspi.tailea056c.ts.net
 (served by Tailscale proxy)
Uptime Kuma	http://192.168.178.69:3001
	http://taylorspi.tailea056c.ts.net:3001

🛠️ Management Commands
# Stop VeraCrypt-dependent services
docker compose -f /home/taylo/docker/docker-compose.yml stop filebrowser resilio

# Start them again
docker compose -f /home/taylo/docker/docker-compose.yml start filebrowser resilio

# Update all containers
docker compose -f /home/taylo/docker/docker-compose.yml pull
docker compose -f /home/taylo/docker/docker-compose.yml up -d

🔒 Notes

filebrowser and resilio depend on /mnt/secure; managed automatically by restore-secure.service.

Home Assistant data lives in /mnt/external/homeassistant.

uptime-kuma monitors all critical web endpoints (Pi-hole, Home Assistant, n8n, etc.).

glances is lightweight and safe to keep running permanently.
