# Systemd Failure Handling Lab (Restart, Rate Limiting, Exit Codes)

## Overview

This scenario demonstrates how `systemd` reacts to **intentional service failures**  
and how failure behavior can be **controlled and rate-limited** using native systemd directives.

The goal is to show practical understanding of:
- Exit codes
- Service failure states
- Automatic restarts
- Restart limits
- How systemd distinguishes between success and failure

This is a **system-level lab**, not an application-level project.

---

## Scenario Description

We implement a service that runs a script which:
- Succeeds only in a certain percentage of executions
- Fails intentionally in the remaining cases
- Exits with proper exit codes (`0` for success, non-zero for failure)

`systemd` is then configured to:
- Restart the service when it fails
- Stop retrying after too many failures in a short time window
- Mark the service as failed when limits are exceeded

---

## Script Logic

The script (Python or Bash) follows this logic:

1. Generate a random number
2. If the number falls within the "success range":
   - Log success
   - Exit with code `0`
3. Otherwise:
   - Log failure
   - Exit with non-zero code (e.g. `1`)

This allows deterministic testing of systemd failure handling.

---

## Why Exit Codes Matter

Systemd **does not parse logs** to determine success or failure.

It relies solely on:
- Process exit codes

| Exit Code | Meaning to systemd |
|----------|-------------------|
| 0 | Success |
| non-zero | Failure |

This makes exit codes a **contract** between the service and systemd.

---

## systemd Service Configuration

The service unit is configured with the following concepts:

### Automatic Restart

- `Restart=on-failure`
- Restarts only when the service exits with a failure code

### Restart Delay

- `RestartSec=5`
- Prevents rapid restart loops

### Failure Rate Limiting

- `StartLimitIntervalSec`
- `StartLimitBurst`

These settings protect the system from:
- Crash loops
- Resource exhaustion
- Uncontrolled retries

When the limit is exceeded:
- systemd stops restarting the service
- service enters a **failed** state

---

## Observed Behavior

Depending on randomness:
- Sometimes the service succeeds on the first run
- Sometimes it fails multiple times and restarts
- When failures happen too frequently:
  - systemd blocks further restarts
  - logs clearly show rate limiting in action

This behavior is verified using:
- `systemctl status`
- `journalctl -u <service>`

---

## Key Takeaways

- systemd is **state-driven**, not log-driven
- Failure handling is declarative, not scripted
- Restart policies are enforced at the init-system level
- Proper exit codes are critical for correct service behavior
- Rate limiting protects system stability

---

## Why This Matters

This lab demonstrates real-world skills used in:
- Production Linux systems
- Service hardening
- Monitoring and reliability engineering
- Debugging unstable services

It shows understanding beyond simply "writing a unit file".

---


## Disclaimer

This project is for educational purposes only.  
Failures are intentional and controlled.