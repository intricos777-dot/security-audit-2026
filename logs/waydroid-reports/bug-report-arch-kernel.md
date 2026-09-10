# Bug Report: Waydroid container stops, HAL missing, cgroup readonly errors on linux-zen 7.2.2

**Host:** Linux 7.2.2-zen1-1-zen  
**Distro:** Garuda Linux (Arch-based)  
**Packages:** waydroid 1.6.3-1, lxc 1:7.0.0-2, linux-zen 7.2.2.zen1-1  
**Waydroid image:** waydroid-image-gapps 20.0_20260428-1

## Summary

Waydroid container starts and reaches `RUNNING` briefly, but `waydroid status` shows `STOPPED` after container start/stop loops. Android HAL `vendor.audio-hal-2-0` is missing, cgroup operations fail with `Read-only file system`, and `lxc.hook.post-stop` fails with exit code 126.

## Steps to Reproduce

1. Install `waydroid` and `waydroid-image-gapps` from Chaotic AUR
2. Run `sudo waydroid init --system_type VANILLA`
3. Run `sudo systemctl enable --now waydroid-container`
4. Run `DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus XDG_RUNTIME_DIR=/run/user/1000 waydroid session start`
5. Observe session starts, then container stops; `waydroid status` shows `STOPPED`

## Logs

### waydroid.log
```
[13:13:48] Starting up container for a new session
...
lxc-start: waydroid: ../src/lxc/conf.c: turn_into_dependent_mounts: 3356 No such file or directory
[13:13:48] RUNNING
[13:14:18] Stopping container
...
lxc-start: waydroid: ../src/lxc/utils.c: run_buffer: 569 Script exited with status 126
lxc-start: waydroid: ../src/lxc/start.c: lxc_end: 1178 Failed to run lxc.hook.post-stop for container "waydroid"
```

### dmesg (relevant lines)
```
init: Could not start service 'vendor.audio-hal-2-0' as part of class 'hal': Cannot find '/vendor/bin/hw/android.hardware.audio@2.0-service': No such file or directory
libprocessgroup: Failed to make and chown /sys/fs/cgroup/uid_1000: Read-only file system
init: createProcessGroup(1000, 81) failed for service 'vendor.audio-hal': Read-only file system
```

### Kernel config
```
# CONFIG_ANDROID_BINDER_IPC is not set
CONFIG_ANDROID_BINDER_IPC_RUST=y
```

## Issues

1. **Missing HAL binary**: `/vendor/bin/hw/android.hardware.audio@2.0-service` not found in rootfs or vendor image. Audio HAL fails to start.
2. **cgroup read-only**: Android init cannot create cgroups under `/sys/fs/cgroup`, causing multiple services to fail `createProcessGroup`.
3. **lxc post-stop hook exits 126**: `/usr/lib/waydroid/lxc/waydroid/hooks/post-stop` returns 126 on shutdown, though this is non-fatal.
4. **Kernel regression risk**: `CONFIG_ANDROID_BINDER_IPC_RUST` is set, classic `CONFIG_ANDROID_BINDER_IPC` is not. Known Waydroid regression on Arch kernels ≥6.18 (github.com/waydroid/waydroid/issues/2157).
5. **Image mismatch**: `waydroid-image-gapps` is installed but init used `--system_type VANILLA`, which may omit GAPPS HALs.

## Expected Behavior

Container boots cleanly, `waydroid status` shows `RUNNING`, audio HAL starts, cgroups created successfully.

## Actual Behavior

Container stops after boot, `waydroid status` shows `STOPPED`, audio HAL missing, cgroup errors logged.

## Additional Notes

- `/dev/binderfs` is present with `anbox-binder`, `anbox-vndbinder`, `anbox-hwbinder`
- `/dev/ashmem` is absent
- `waydroid session start` requires explicit `DBUS_SESSION_BUS_ADDRESS` and `XDG_RUNTIME_DIR`
- Waydroid image path is `/usr/share/waydroid-extra/images` (images_path in cfg)
- `waydroid status` shows `IP address: UNKNOWN`

## Reproducibility

Always reproducible on current kernel/config/image combination.

## Possible Fixes / Workarounds

1. Reinitialize with GAPPS image: `sudo waydroid init --system_type GAPPS`
2. Switch to linux-lts kernel with classic binder IPC
3. Report kernel regression to Arch kernel maintainers
4. Ensure cgroupv2 delegation is enabled or disable cgroup restrictions in LXC config
