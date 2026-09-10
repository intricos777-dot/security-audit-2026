# Waydroid Setup — Coordination Brief

## Current State
- **Waydroid status:** Session: RUNNING, Container: RUNNING
- **Wayland display:** wayland-0
- **Kernel:** linux-zen 7.2.2-zen1-1-zen
- **Waydroid:** 1.6.3-1, LXC: 1:7.0.0-2

## Verified
- waydroid init completed
- systemd container service enabled/active
- Container boots and stays RUNNING
- HAL services running (except audio-hal)
- Apps visible (Files, Messages, Play Store, Recorder, Google)

## Outstanding Issues
1. Missing audio HAL: vendor.audio-hal-2-0
2. cgroup readonly errors
3. lxc.hook.post-stop exits 126
4. rust_binder ENOSPC errors
5. IP address UNKNOWN
6. Known kernel regression with CONFIG_ANDROID_BINDER_IPC_RUST

## Reports
- /home/sin/waydroid-reports/setup-report.txt
- /home/sin/waydroid-reports/diagnosis.txt
- /home/sin/waydroid-reports/bug-report-arch-kernel.md

## Tracks for Open Agents
### Track A: Image investigation
Investigate why vendor.audio-hal-2-0 is missing from VANILLA image vs GAPPS.
### Track B: cgroup/LXC config
Check if cgroup delegation or LXC config can resolve readonly errors.
### Track C: Network/IP
Diagnose why waydroid status shows UNKNOWN IP despite dnsmasq running.
### Track D: Kernel regression
Draft upstream report for linux-zen/CONFIG_ANDROID_BINDER_IPC_RUST issue.

## Session Notes
- Interactive pts/1: Hermes CLI
- Interactive pts/2: Hermes CLI
- Interactive pts/3: OpenCode TUI
- Interactive pts/5: OpenCode TUI
