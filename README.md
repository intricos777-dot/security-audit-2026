# Security Logs Archive — Public Audit 2026

**Owner:** sin (intricos777-dot)  
**System:** Garuda Linux / Arch-based, linux-zen 7.2.2-zen1-1-zen  
**Period covered:** 2026-09-01 through 2026-09-10  
**Audit performed:** 2026-09-10 by Hermes Agent

---

## What's in this repo

Full collection of system logs, security monitoring output, and diagnostic reports from a Garuda Linux desktop system, archived for public review. The goal: transparent record of what the logs show regarding network attacks, intrusion attempts, and system security events.

### Directory structure

```
.
├── findings/
│   └── README-findings.md    # Detailed audit findings and analysis
├── logs/
│   ├── security-logs/        # Attacks log, audit rules, crontab, logrotate conf
│   ├── waydroid-reports/     # Full Waydroid diagnostics, bug reports, tracks
│   ├── project-logs/         # Self-heal/debug logs from GitHub projects
│   └── system-journal/       # (journal output — see below)
├── README.md                 # This file
└── SECURITY.md               # Security policy and disclosures
```

### Key files

| File | Description |
|------|-------------|
| `logs/security-logs/attacks.log` | Attack monitor output — suspicious process detection (2026-09-08) |
| `logs/security-logs/audit.rules` | Configured auditd rules for auth, file integrity, process monitoring |
| `logs/security-logs/crontab` | Cron configuration for attack monitoring (every 5 min) |
| `logs/waydroid-reports/*.md` | Waydroid setup, diagnosis, bug reports, issue tracks |
| `logs/waydroid-reports/*.txt` | Waydroid diagnostic text reports |
| `logs/project-logs/` | Self-heal and debug logs from projects under github-intricos777 and Projects |

---

## Security findings summary

**See `findings/README-findings.md` for the full detailed report.**

**Headline:** No confirmed external network intrusion found in the available logs. The logs show internal crypto mining processes, firewall instability, a disabled SSH daemon, and auditd rules that are configured but not loaded.

Key points:
- **attacks.log** flags mining processes (SRBMiner, minerd) — these are local, user-launched miners, not external attacks.
- **No `/var/log/auth.log` exists** — SSH auth activity not locally logged. SSH daemon is disabled and inactive.
- **Auditd rules are configured but not loaded** — file integrity and privilege escalation monitoring is inactive.
- **Martian source packets** from 10.128.128.128 (the system's DNS server) — likely benign routing artifact, blocked by firewall.
- **firewalld** is unstable — Python TypeError crash, multiple restart loops.
- **No suspicious network listeners** — only Hermes (localhost:4123), mining stratum ports, and kdeconnect.

---

## System context

This is a personal desktop system used for:
- Software development (multiple GitHub projects)
- Gaming (Steam, Proton, Waydroid for Android apps)
- Crypto mining (NiceHash, via local scripts)
- Security/biometric experimentation (bde-daemon)
- AI agent workflows (Hermes, OpenCode)

The system uses KDE Plasma on Wayland, firewalld/nftables for firewall, and LUKS full-disk encryption.

---

## Limitations

- **No packet capture** — only firewall drop logs, not full traffic.
- **Journal covers recent boots only** — older boot logs not available.
- **Auditd inactive** — no file integrity or execve auditing data.
- **No auth.log** — SSH/auth events not logged locally.
- **Self-reported** — this archive is assembled by the system owner's AI agent from local logs.

---

## External references

- GitHub: https://github.com/intricos777-dot
- Waydroid issues tracked at: https://github.com/intricos777-dot/waydroid-issues
- Arch bug #173, Waydroid issue #2157 (kernel binder regression)

---

*As above, so below — the logs reflect what the system saw, nothing more.*
