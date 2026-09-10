# Waydroid Final Status — 2026-09-03

## Host Stability
- `systemctl --failed`: 0 failed units
- dmesg errors are limited to Waydroid container/Android init lines
- No host boot regressions from our edits
- Firewall changes reverted to original state
- LXC config restored from backup; only investigation edits were made

## Current Runtime
- waydroid status: Session STOPPED, Container STOPPED
- waydroid-container.service: enabled, active
- cgroup readonly errors persist
- DHCP lease file empty; IP UNKNOWN
- rust_binder ENOSPC noise present on current kernel

## Actionable Paths
1. Do not install linux-lts515: kernel 5.15.219 may lack needed ZEN/ZFS/hardware support
2. Custom kernel with classic binder required for clean Waydroid runtime
3. Upstream image bug for audio HAL should be reported

## Reports
- /home/sin/waydroid-reports/setup-report.txt
- /home/sin/waydroid-reports/diagnosis.txt
- /home/sin/waydroid-reports/bug-report-arch-kernel.md
- /home/sin/waydroid-reports/track-a-image-hal.md
- /home/sin/waydroid-reports/track-b-cgroup.md
- /home/sin/waydroid-reports/track-c-network.md
- /home/sin/waydroid-reports/track-d-kernel-regression.md
- /home/sin/waydroid-reports/final-status.md
