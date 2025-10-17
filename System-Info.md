# 🖥️ Raspberry Pi System Information (TaylorsPi)

## 🧩 System Overview
- **Hostname:** TaylorsPi  
- **Kernel:** Linux 6.12.47+rpt-rpi-v8  
- **Architecture:** aarch64 (64-bit)  
- **Operating System:** Debian GNU/Linux 13 (Trixie)  
- **Version ID:** 13.1  
- **Codename:** trixie  
- **Vendor:** Debian  
- **Platform:** Raspberry Pi (ARM Cortex-A72)  
- **Build:** #1 SMP PREEMPT Debian 1:6.12.47-1+rpt1 (2025-09-16)

---

## ⚙️ CPU Details
- **Model:** ARM Cortex-A72 (r0p3)  
- **Cores:** 4 (1 thread per core)  
- **Clock Range:** 600 MHz – 1800 MHz  
- **BogoMIPS:** 108.00  
- **CPU Scaling:** 100%  
- **Flags:** `fp`, `asimd`, `evtstrm`, `crc32`, `cpuid`  
- **Caches:**  
  - L1d: 128 KiB ×4  
  - L1i: 192 KiB ×4  
  - L2: 1 MiB  
- **NUMA Nodes:** 2 (node0 CPUs 0–3, node1 CPUs 0–3)

---

## 🧱 CPU Vulnerabilities / Mitigations
| Vulnerability | Status |
|----------------|--------|
| Gather data sampling | Not affected |
| Indirect target selection | Not affected |
| Itlb multihit | Not affected |
| L1tf | Not affected |
| Mds | Not affected |
| Meltdown | Not affected |
| Speculative store bypass | **Vulnerable** |
| Spectre v1 | Mitigated (__user pointer sanitization) |
| Spectre v2 | **Vulnerable** |
| Others | Not affected |

---

## 🧮 Memory
| Type | Total | Used | Free | Shared | Buff/Cache | Available |
|------|------|------|------|--------|-------------|-----------|
| RAM  | 3.7 GiB | 572 MiB | 91 MiB | 20 MiB | 3.1 GiB | 3.1 GiB |
| Swap | 2.0 GiB | 0B | 2.0 GiB | – | – | – |

---

## 💽 Storage Overview (`df -h`)
| Filesystem | Size | Used | Avail | Use% | Mounted on |
|-----------|------|------|-------|------|------------|
| /dev/sda2 | 29G | 5.4G | 23G | 20% | `/` |
| /dev/sda1 | 510M | 74M | 437M | 15% | `/boot/firmware` |
| /dev/sdb1 | 4.6T | 353G | 4.3T | 8% | `/mnt/external` |
| tmpfs | 1.9G | 8.4M | 1.9G | 1% | `/tmp` |
| others | (various system mounts) | – | – | – | – |

---

## 🧰 Block Devices (`lsblk`)
| Device | Size | FSType | Label | Mountpoint |
|-------|------|--------|-------|-----------|
| **loop0** | 2G | swap | – | – |
| **loop1** | 200G | – | – | – |
| └─veracrypt1 | 200G | exfat | – | (mounts dynamically at `/mnt/secure`) |
| **sda1** | 512M | vfat | bootfs | `/boot/firmware` |
| **sda2** | 29.5G | ext4 | rootfs | `/` |
| **sdb1** | 4.5T | ntfs | My Passport | `/mnt/external` |
| **zram0** | 2G | swap | zram0 | [SWAP] |

---

## 🧾 Notes
- The VeraCrypt container file lives on the **My Passport** drive:
  `/mnt/external/My files.hc`
  → When mounted, it appears at `/mnt/secure`.

- Root filesystem: 29 GB on internal storage (`/dev/sda2`)  
- External storage: 4.5 TB NTFS volume (`My Passport`) used for Syncthing backups and encrypted container.  
- Swap configured via both `loop0` and `zram0`.

---

### 📦 Summary
This Raspberry Pi runs Debian 13 (Trixie) on ARM64 architecture with a quad-core Cortex-A72 CPU, 4 GB RAM, and a large external NTFS drive for backups. It hosts services like Docker, Syncthing, and VeraCrypt, managed under the `taylo` user.
