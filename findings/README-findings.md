# Security Audit Findings — September 2026

## System
- **Host:** Linux 7.2.2-zen1-1-zen (Garuda Linux / Arch-based)
- **Kernel:** linux-zen 7.2.2-zen1-1-zen (7.2.3.zen1-2 installed, reboot pending)
- **User:** sin (uid 1000)
- **Audit period:** 2026-09-01 through 2026-09-10

---

## Executive Summary

**No confirmed external network intrusion detected in the available logs.**

The logs show:
1. **Internal crypto miners** running under the user account (SRBMiner, minerd) — these are local processes, not external attacks. Detected by the attack monitor on 2026-09-08.
2. **Martian source packets** from 10.128.128.128 — these appear to be DNS responses (port 53) from the system's own configured DNS server, not attacks.
3. **No SSH auth log exists** (`/var/log/auth.log` is absent) — so SSH brute force attempts cannot be assessed from local logs. SSH daemon is installed but **disabled and inactive**.
4. **Auditd rules are configured** but **no rules are currently loaded** (`auditctl -l` returns empty) — so file integrity and privilege escalation monitoring is not active.
5. **Firewall (firewalld)** had multiple restart loops and a Python TypeError crash, then was stopped. Currently started but unstable across recent boots.
6. **bde-daemon** (bioelectrical biometrics) is in a crash loop — `Read-only file system` on `/var/lib/bde-daemon`. Restart counter at 126+ as of 11:19 PDT.
7. **No suspicious external connections** found in `ss` output — only kdeconnect, the node Hermes process on localhost:4123, and the mining pool stratum ports (21550, 1716).

---

## Detailed Findings

### 1. Attacks.log — Suspicious Process Detection (2026-09-08)

File: `logs/security-logs/attacks.log`

Three processes flagged on 2026-09-08T13:45:

| PID | Process | Type |
|-----|---------|------|
| 1143 | SRBMiner-MULTI (autolykos2, nicehash) | miner |
| 1144 | minerd (scrypt, ltc.viabtc.io) | miner |
| 1145 | steam-pause-watcher.sh | miner watchdog |
| 1189 | minerd (scrypt, zpool.ca) | miner |

All are mining-related. These are **local processes** started by the user's own scripts in `/home/sin/miners/`. The attack monitor flagged them because they match "miner" and "stratum" patterns — but they are not external intrusions. They are self-run mining operations.

### 2. Martian Source Packets (2026-09-10, multiple boots)

Kernel log shows repeated:
```
IPv4: martian source (src=10.128.128.128, dst=255.255.255.255, dev=wlo1)
```

And nftables DEVICE-GUARD blocks:
```
DEVICE-GUARD-BLOCK: IN=wlo1 ... SRC=10.128.128.128 DST=10.245.106.176 PROTO=UDP SPT=53 ...
```

**Analysis:** 10.128.128.128 is the system's configured DNS server (`/etc/resolv.conf`). These are DNS response packets being flagged as martian because they arrive on wlo1 with a source that the kernel considers unexpected. This is likely a routing/NAT artifact from the network setup, not an attack. The DEVICE-GUARD firewall is correctly blocking unsolicited inbound DNS responses.

### 3. Firewall instability

journalctl shows firewalld crashed with:
```
TypeError: FirewallD.addService() got multiple values for argument 'sender'
```

This is a Python/firewalld version incompatibility. firewalld was stopped and restarted multiple times across 4 different boot sessions on 2026-09-10. Currently running but may crash again.

### 4. SSH — No exposure

- `sshd.service`: **disabled, inactive (dead)**
- No `/var/log/auth.log` exists
- No SSH brute force attempts can be detected from local logs
- SSH socket exists (systemd-ssh-generator) but daemon is not running

**Conclusion:** The system is not exposing SSH to the network. No SSH-based attacks possible while daemon is offline.

### 5. Auditd — Not active

- `/home/sin/security-logs/audit.rules` contains comprehensive rules (auth, file integrity, process execution, privilege escalation)
- `sudo auditctl -l` returns **No rules** — rules are not loaded
- Attack monitor script (`/home/sin/bin/attack-monitor.py`) exists but relies on auditd for many checks
- Cron job runs every 5 minutes but audit-dependent checks return nothing

**Gap:** File integrity monitoring and privilege escalation detection are not currently functioning.

### 6. bde-daemon crash loop

```
OSError: [Errno 30] Read-only file system: '/var/lib/bde-daemon'
```

Restart counter at 126+ (as of 11:19 PDT, counting up ~1/second). This is a local service failure, not an attack. The `/var/lib` filesystem appears to be read-only, possibly from a previous boot issue or filesystem state.

### 7. Network connections (ss output)

```
tcp  LISTEN  127.0.0.1:4123    — node-MainThread (Hermes)
tcp  LISTEN  0.0.0.0:21550     — SRBMiner API (mining)
tcp  LISTEN  *:1716            — kdeconnectd
udp  UNCONN  *:5353           — kdeconnectd (mDNS)
udp  UNCONN  *:1716           — kdeconnectd
```

No unexpected listeners. No reverse shells. No backdoor ports.

### 8. Project logs

All project self-heal and debug logs (polsia-cli, ai-autonomy-framework, crypto-bot, local-emergency-patrol) show routine self-debug/self-heal cycles from 2026-08-08. No signs of compromise in these logs.

---

## What's Missing / Not Assessable

1. **No `/var/log/auth.log`** — SSH/auth activity not logged locally. Cannot assess SSH brute force or unauthorized login attempts.
2. **Auditd rules not loaded** — file integrity and privilege escalation monitoring inactive.
3. **No external network capture** — only local firewall drop logs available. Cannot see what reached the network interface before firewall filtering.
4. **Journal only covers current boot + recent boots** — older boots' logs are not available for review.
5. **martian packets** — without packet capture, cannot definitively determine if 10.128.128.128 behavior is benign routing or spoofed traffic.

---

## Recommendations

1. **Enable auditd** — load the existing audit.rules: `sudo auditctl -R /home/sin/security-logs/audit.rules`
2. **Enable SSH logging** — if SSH is needed, ensure `LogLevel VERBOSE` and `auth.log` persistence. If not needed, keep sshd disabled (current state is safer).
3. **Fix firewalld** — the Python TypeError suggests a package version mismatch. Reinstall or upgrade firewalld.
4. **Fix bde-daemon** — make `/var/lib/bde-daemon` writable or disable the service if not needed.
5. **Investigate 10.128.128.128** — confirm this is the legitimate DNS server. If it's unsolicited, investigate network routing.
6. **Consider capturing network traffic** — tcpdump on wlo1 for a period would give definitive evidence of external scan/attack attempts.

---

*Generated: 2026-09-10 by Hermes Agent security audit*
