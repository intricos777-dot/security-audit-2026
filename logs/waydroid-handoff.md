# Waydroid Setup Handoff Notes (last updated: this session)

## Goal
Run Android apps (Mode Earn from Play Store) inside Waydroid on this system
(Arch/Garuda, i5-12450H, Wayland).

## What is installed already
- `waydroid` 1.6.3 (official extra repo)
- `waydroid-image-gapps` 20.0 (chaotic-aur gapps image with Google Play)
  - images at /usr/share/waydroid-extra/images (system.img + vendor.img)
- `python-pyclip` (clipboard support for waydroid session)

## Config that is set (persists across reboot)
- /var/lib/waydroid/waydroid.cfg:
  - suspend_action = null   (keeps container running when idle)
  - images_path = /usr/share/waydroid-extra/images
  - binder = anbox-binder, vendor_type = MAINLINE
- firewalld (permanent, survives reboot):
  - interface waydroid0 added to zone=trusted
  - masquerade enabled on trusted
  - (command used: firewall-cmd --zone=trusted --add-interface=waydroid0 --permanent
     and --zone=trusted --add-masquerade --permanent)
- linux-zen kernel has binder built-in (CONFIG_ANDROID_BINDER_IPC_RUST=y;
  legacy CONFIG_ANDROID_BINDER_IPC is NOT set - it was removed upstream)

## Status at last session: WAYDROID IS BROKEN/UNSTABLE
- Container DID fully boot + show Android desktop + Play Store earlier
  ("Android with user 0 is ready"), but:
  1. NETWORKING NEVER WORKED: container eth0 got NO IPv4 (no DHCP lease from
     host dnsmasq on waydroid0/192.168.240.1). IP status shows "UNKNOWN".
     Android IpClient/EthernetTracker sees eth0 up but networkAgent stays null /
     link-local only -> no internet -> cannot download from Play Store.
  2. INTERMITTENT BINDER FAILURE: kernel rust_binder logs
     "rust_binder: Failed to allocate buffer... ENOSPC" on large transactions.
     This can crash Android's hwservicemanager/servicemanager, causing
     "Hardware service is not even started" and container teardown.
  3. Extensive manual tinkering (repeated systemctl stop/start, background
     starts, pkill) left runtime state corrupted; container stopped booting
     reliably.

## Plan after reboot (fresh start)
1. Do NOT rely on /tmp scripts (they are wiped on reboot).
2. Confirm firewall still OK:
     sudo firewall-cmd --zone=trusted --list-all
   (should show interface waydroid0 + masquerade yes)
3. Start container via systemd (canonical path):
     sudo systemctl start waydroid-container.service
   Confirm it reaches RUNNING:
     sudo lxc-info -P /var/lib/waydroid/lxc -n waydroid -sH
4. Start session as THIS user with proper env (NOT under sudo):
     export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus
     export WAYLAND_DISPLAY=wayland-0
     export XDG_RUNTIME_DIR=/run/user/1000
     waydroid session start
5. If binder failures persist (rust_binder ENOSPC) -> real kernel limitation;
   legacy binder removed upstream so no kernel switch fixes it. May need to
   try waydroid with vendor_type=MAINLINE vs the vanilla (non-gapps) image,
   or reduce large binder transactions. This is NOT fixable by rebuilding
   waydroid source or a custom kernel binder.
6. If networking (DHCP) persists: the container eth0 must obtain an IPv4 from
   host dnsmasq. Debug from host: check `pgrep -a dnsmasq` is bound to
   waydroid0, verify veth is attached (`ls /sys/class/net/waydroid0/brif/`),
   and from inside container: ifconfig eth0.

## Rejected approaches (do NOT re-pursue)
- Bluestacks via Wine -> does not work
- Custom kernel binder driver -> legacy binder removed upstream, impractical
- Rebuilding waydroid from source -> not the cause
- "In-app network bridge" built from scratch -> not the cause

## Note on "Hermes"
hermes-pm was uninstalled this session. No Hermes tool exists to pass logs to.
No waydroid GitHub repo was created in this environment. Do not expect a
Hermes integration.
