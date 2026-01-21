# Systemd Monitoring Lab (LPIC-2 Practical Scenario)

This document describes a complete system monitoring scenario using a custom bash script and systemd units in a virtualized lab environment as part of LPIC-2 practical exercises.  
The focus of this lab is to understand creating scripts, configuring systemd services and timers, logging behavior, and verifying functionality.

**Environment:**  
- systemd-based: Ubuntu 
- Platform: Virtual Machines (isolated lab environment)  

---

## 1. Problem Definition

The goal of this scenario was to:

- Create a custom system monitoring script (`sysmon.sh`) that collects system information such as host details, disk usage, and timestamped logs.
- Store collected logs in a dedicated directory (`/var/log/sysmon.log/`) for later analysis.
- Configure a **systemd service** to execute the script.
- Configure a **systemd timer** to run the service periodically.
- Understand systemd unit structure, journal logging, and execution flow.
- Observe errors and corrections while building the system.

---

## 2. Implementation Steps

### 2.1 Script Creation

1. Create the bash script `sysmon.sh` in the user’s home directory:
   - Contents include collecting:
     - Timestamp: `TIME=$(date '+%Y-%m-%d %H:%M:%S')`
     - Host information: `hostnamectl`
     - Disk usage: `df -h`
     - Logging output to file: `/var/log/sysmon.log`
2. Make the script executable:

```bash
chmod +x /usr/local/bin/sysmon.sh
```

**Notes from testing/errors:**
- Initially, variables in the script were interpreted as commands (e.g., `LOG_FILE: command not found`). Corrected by using proper assignment syntax without spaces around `=` and using `$LOG_FILE` for redirection.
- Incorrect `bash` shebang: `/bin/bash/` → corrected to `/bin/bash`
- The script was successfully executed standalone before linking to systemd.

---

### 2.2 Systemd Service Setup

1. Create the service unit file `/etc/systemd/system/sysmon.service`:

```ini
[Unit]
Description=Custom System Monitoring Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/sysmon.sh
```

2. Reload systemd to recognize the new service:

```bash
systemctl daemon-reload
```

3. Start and verify the service:

```bash
systemctl start sysmon.service
systemctl status sysmon.service
journalctl -u sysmon.service
```

**Notes / errors:**
- Initial error: `usr/local/bin/sysmon.sh : not a directory (os error 20)` → caused by trailing slash in shebang. Fixed `/bin/bash/` → `/bin/bash`.
- The service worked successfully after correcting the script path.

---

### 2.3 Systemd Timer Setup

1. Create the timer unit file `/etc/systemd/system/sysmon.timer`:

```ini
[Unit]
Description=Run sysmon service every 2 minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=2min

[Install]
WantedBy=timers.target
```

2. Reload systemd and start the timer:

```bash
systemctl daemon-reload
systemctl start sysmon.timer
systemctl enable sysmon.timer
systemctl status sysmon.timer
```

**Notes / errors:**
- Initial typo: `[Install]` section used `Wantedby` instead of `WantedBy` → caused enable failure. Corrected spelling.
- Timer now correctly triggers `sysmon.service` every 2 minutes.

---

### 2.4 Log Analysis

1. Check the logs written by the script:

```bash
cat /var/log/sysmon.log
```

2. Verify service and timer activity using `journalctl`:

```bash
journalctl -u sysmon.service
journalctl -u sysmon.timer
journalctl -f
```

**Observations:**
- Each execution logs host info, timestamp, and disk usage.
- Errors in execution or variable assignment are visible in `journalctl`.
- Shows interaction between systemd, the timer, and script execution.

---

### 2.5 Key Learnings and Notes

- Bash script variables must be correctly defined (`VAR=value`) with no spaces around `=`.  
- Paths in `ExecStart` must point to executable scripts.  
- `systemd.service` units require `Type=oneshot` for scripts that run and exit.  
- `systemd.timer` units require a correct `[Install]` section (`WantedBy=timers.target`) to be enabled.  
- Logs can be monitored via `/var/log/sysmon.log` and `journalctl` for both service and timer units.  
- This scenario illustrates how systemd services and timers automate tasks, track execution, and integrate with Linux logging mechanisms.

**Security/Best Practice Notes:**
- Run scripts from secure directories (e.g., `/usr/local/bin/`) and verify ownership/permissions.
- Combine logging with monitoring to detect abnormal execution.
- Timer-based automation can replace cron in systemd-based environments.
- Layer additional security if needed (file permissions, firewall, SELinux).

---

### 3. Evidence and Documentation

**Recommended screenshots for the verification:**

- Script contents and execution: `/usr/local/bin/sysmon.sh`
- Service unit file: `/etc/systemd/system/sysmon.service`
- Timer unit file: `/etc/systemd/system/sysmon.timer`
- `systemctl status` outputs for both service and timer
- Logs from `/var/log/sysmon.log`
- `journalctl -u sysmon.service` and `journalctl -u sysmon.timer` outputs

These demonstrate that the systemd service and timer function as intended and show logging behavior.

---


This document and the attached logs/screenshots provide a **complete, traceable lab scenario** , demonstrating proficiency with Linux systemd services, timers, and automation.