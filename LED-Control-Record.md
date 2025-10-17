Overview

Purpose: Nightly LED off; morning restore to normal behavior (green ACT blinks on activity).

User / Host: taylo on TaylorsPi.

Script location: /usr/local/bin/ledctl.sh (root-owned, executable). This matches where you keep custom system scripts (e.g., secure shutdown).

Files
1) LED control script

Path: /usr/local/bin/ledctl.sh
Mode: 755 (executable)
Owner: root:root

Current version (supports on/off/restore):

#!/bin/bash
set -euo pipefail

# Candidate LED paths (covers ACT/PWR and led0/led1 variants)
ACT_CANDIDATES=(/sys/class/leds/ACT /sys/class/leds/led0)
PWR_CANDIDATES=(/sys/class/leds/PWR /sys/class/leds/led1)

_exists() { [ -d "$1" ]; }
_pick() { for L in "$@"; do _exists "$L" && { echo "$L"; return 0; }; done; return 1; }

set_fixed() { # on|off
  local state="$1" val=0
  [ "$state" = "on" ] && val=1
  for L in "$(_pick "${ACT_CANDIDATES[@]}")" "$(_pick "${PWR_CANDIDATES[@]}")"; do
    [ -n "${L:-}" ] || continue
    echo none | tee "$L/trigger" >/dev/null
    echo "$val" | tee "$L/brightness" >/dev/null
  done
}

_supports_trigger() { local L="$1" trig="$2"; grep -qw -- "$trig" "$L/trigger"; }

restore_leds() {
  # ACT (green): prefer mmc0/mmc1, fallback heartbeat, then default-on
  local ACT="$(_pick "${ACT_CANDIDATES[@]}")"
  if [ -n "${ACT:-}" ]; then
    if _supports_trigger "$ACT" mmc0; then echo mmc0 | tee "$ACT/trigger" >/dev/null
    elif _supports_trigger "$ACT" mmc1; then echo mmc1 | tee "$ACT/trigger" >/dev/null
    elif _supports_trigger "$ACT" heartbeat; then echo heartbeat | tee "$ACT/trigger" >/dev/null
    else echo default-on | tee "$ACT/trigger" >/dev/null; fi
  fi
  # PWR (red): prefer input, fallback default-on
  local PWR="$(_pick "${PWR_CANDIDATES[@]}")"
  if [ -n "${PWR:-}" ]; then
    if _supports_trigger "$PWR" input; then echo input | tee "$PWR/trigger" >/dev/null
    else echo default-on | tee "$PWR/trigger" >/dev/null; fi
  fi
}

usage(){ echo "Usage: $0 on|off|restore" >&2; exit 2; }

case "${1:-}" in
  on|off)  set_fixed "$1" ;;
  restore) restore_leds ;;
  *)       usage ;;
esac

Scheduling (root crontab)

Editor: sudo crontab -e
Current entries:

# ENV is root; use absolute PATHs
SHELL=/bin/bash
PATH=/usr/sbin:/usr/bin:/sbin:/bin

# Turn LEDs OFF every night at 22:00
0 22 * * * /usr/local/bin/ledctl.sh off

# In the morning, RESTORE normal triggers (ACT -> mmc0, PWR -> input/default-on)
0 7  * * * /usr/local/bin/ledctl.sh restore

Verify / Troubleshoot
One-time checks
# Show triggers (current in [brackets])
for L in /sys/class/leds/*; do echo "=== $L ==="; cat "$L/trigger"; done

# Test: OFF then RESTORE
sudo /usr/local/bin/ledctl.sh off
sleep 2
sudo /usr/local/bin/ledctl.sh restore

Integrity & perms
# Ensure script is root-owned + executable
ls -l /usr/local/bin/ledctl.sh

# Save a checksum so you know if it changes later
sha256sum /usr/local/bin/ledctl.sh

Cron audit (to catch old jobs elsewhere)
sudo crontab -l
crontab -l
sudo grep -R "led" /etc/cron* 2>/dev/null
systemctl list-timers --all | grep -i led || true

Boot config (in case of overrides)
grep -i led /boot/firmware/config.txt

🧩 Ethernet Port LED Status (Hardware Limitation)
[... unchanged content ...]
