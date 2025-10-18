# Taylor’s Pi System Profile

(Updated 2025-10-17 20:33 CEST)

# Taylor’s Pi — System Profile

## 🧩 Hardware / OS
| Component | Details |
|------------|----------|
| **Device** | Raspberry Pi 4 Model B (4 GB) |
| **Architecture** | ARM 64-bit (aarch64) |
| **OS** | Debian GNU/Linux 13 (trixie) |
| **Kernel** | 6.12.47+rpt-rpi-v8 (2025-09-16) |
| **User** | taylo |
| **Hostname** | TaylorsPi |
| **Network IP (eth0)** | 192.168.178.69 |
| **Time Zone** | Europe/Berlin |

---

## 🔐 VeraCrypt
| Field | Value |
|-------|--------|
| Container file | `/mnt/external/My files.hc` |
| Mount point | `/mnt/secure` |
| Filesystem | exFAT (uid=1000,gid=1000,fmask=0077,dmask=0077) |
| Mount binary | `/usr/bin/veracrypt` |
| Service integration | `restore-secure.service` (one-shot at boot) |
| Backup script | `/home/taylo/veracrypt_backup.sh` (CRON: Sun 02:00) |
| Password storage | [REDACTED] |

---

## 🔄 Syncthing
| Item | Value |
|------|--------|
| Service unit | `/usr/lib/systemd/system/syncthing@.service` → `syncthing@taylo.service` |
| Config path | `/home/taylo/.local/state/syncthing/` |
| Web GUI | `https://127.0.0.1:8384` (localhost only) |
| TLS | Enabled |
| API key | [REDACTED] |
| Folder ID | `veracrypt-backup` |
| Folder path | `/mnt/external` |
| Folder type | Send-only |
| Rescan interval | 0 (manual) |

---

## 🐳 Docker Compose
| Item | Value |
|------|--------|
| Compose file | `/home/taylo/docker/docker-compose.yml` |
| Engine version | 28.5.1 (driver overlay2) |
| Compose checksum | `8d5d02f7aeb8926039d8ed2ba01fb7c1b0bbdae6431adacba853590f3038ee16` |
| Restart policy | unless-stopped |

### Active Services
| Service | Ports | Notes |
|----------|--------|-------|
| **filebrowser** | 8080 → 80 | Exposes `/mnt/secure:/srv` |
| **resilio** | 8888 / 55555 | Sync folder at `/mnt/secure:/sync` |
| **pihole** | 53 / 80 | DNS + ad-block, host network |
| **portainer** | 9443 / 9000 | Docker management UI |
| **homeassistant** | 8123 | LAN & Tailscale reachable |
| **n8n** | 5678 | Tailscale HTTPS reverse-proxy (`/`) |
| **uptime-kuma** | 3001 | Internal monitoring for services |
| **glances** | N/A | System resource monitor |

---

## 🧰 Boot Restore Script
| Item | Value |
|------|--------|
| Path | `/usr/local/bin/restore_secure.sh` |
| Unit | `/etc/systemd/system/restore-secure.service` |
| Trigger | `multi-user.target` |
| Dependencies | `/mnt/external`, `docker.service` active, `/mnt/external/My files.hc` exists |
| Behavior | Mounts VeraCrypt → starts Docker services (`filebrowser`, `resilio`) |
| Password input method | `systemd-ask-password` (fallback to `read -s` if TTY) |

---

## 🌐 Network / DNS / Tailscale
| Item | Value |
|------|--------|
| LAN Gateway | 192.168.178.1 |
| DNS Primary | Pi-hole (192.168.178.69) |
| DNS Secondary | Cloudflare (1.1.1.1) |
| Tailscale IPv4 | 100.108.200.53 |
| Tailscale Hostname | `taylorspi.tailea056c.ts.net` |
| Tailscale Serve Mapping | `https://taylorspi.tailea056c.ts.net` → `127.0.0.1:5678` (n8n) |

---

## 🕒 Scheduled Tasks
| Time | Action |
|------|---------|
| Daily 22:00 | LED off (`/usr/local/bin/ledctl.sh off`) |
| Daily 07:00 | LED restore (`/usr/local/bin/ledctl.sh restore`) |
| Sunday 02:00 | VeraCrypt backup (`/home/taylo/veracrypt_backup.sh`) |

---

## 🗝️ Summary
* Root now on `/dev/sda2 (newroot)`; boot on `/dev/sdb1 (bootfs)`.  
* `restore-secure.service` runs successfully with `systemd-ask-password`.  
* All Docker containers verified healthy.  
* Syncthing stopped manually (can be re-enabled).  
* LED and backup cron jobs confirmed.  

---

*(Source: Live audit 2025-10-17 20:33 CEST)*

