# System Information — TaylorsPi

## 🧩 System Overview
| Item | Value |
|------|-------|
| **Hostname** | TaylorsPi |
| **Architecture** | ARM64 (aarch64) |
| **Model** | Raspberry Pi 4 — Cortex-A72 × 4 @ 1.8 GHz |
| **OS** | Debian GNU/Linux 13 (trixie) |
| **Kernel** | Linux 6.12.47+rpt-rpi-v8 #1 SMP PREEMPT (2025-09-16) |
| **User** | taylo |
| **Machine ID** | bf2feb5960b7433bb846bcf6c14cf1a3 |

---

## ⚙️ CPU & Memory
| Item | Value |
|------|-------|
| **Cores** | 4 (Cortex-A72) |
| **CPU min MHz** | 600 |
| **CPU max MHz** | 1800 |
| **Total RAM** | 3.7 GiB |
| **Available RAM** | ≈ 2.5 GiB |
| **Swap (zram0)** | 2 GiB enabled |

---

## 💽 Storage / Boot
| Device | Label | FSType | Mount Point | Size | Notes |
|---------|--------|--------|-------------|------|-------|
| `/dev/sda2` | `newroot` | ext4 | `/` | 79 G (13 used) | **Current root filesystem** |
| `/dev/sdb1` | `bootfs` | vfat | `/boot/firmware` | 510 M (74 used) | Boot partition |
| `/dev/sda1` | `My Passport` | ntfs3 | `/mnt/external` | 4.5 T (353 used) | External drive |
| `/dev/mapper/veracrypt1` | — | exfat | `/mnt/secure` | 200 G (121 used) | Mounted VeraCrypt volume |
| `/dev/zram0` | zram0 | swap | — | 2 G | Active swap |

Root UUID → `3a8190c8-1598-41c0-b303-d1f807c93002`  
Boot UUID → `1C94-4EC3`

---

## 🧾 FSTAB
```text
proc            /proc           proc    defaults          0 0
UUID=4355-12FC  /mnt/storage    vfat    defaults,uid=1000,gid=1000,umask=0002,nofail,x-systemd.device-timeout=10 0 0
UUID=4A8C7C7B8C7C637D /mnt/external ntfs3 uid=1000,gid=1000,umask=002,defaults,nofail,x-systemd.automount 0 0
UUID=3a8190c8-1598-41c0-b303-d1f807c93002 / ext4 defaults,noatime 0 1
UUID=1C94-4EC3 /boot/firmware vfat defaults 0 2

🔐 VeraCrypt

Container path: /mnt/external/My files.hc

Mount point: /mnt/secure

FSType: exfat (uid/gid 1000)

Mount helper: /usr/bin/veracrypt

Service integration: restore-secure.service (one-shot at boot)

🗂️ Summary

System upgraded to Debian 13 (trixie) on new 80 GB root partition (newroot).

Previous root (/dev/sdb2) retained but inactive.

External 4.5 TB drive auto-mounts at /mnt/external.

VeraCrypt container and mount paths unchanged.

All Docker services running; Syncthing temporarily inactive.
