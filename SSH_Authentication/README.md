# SSH Hardening Lab (LPIC-2 Practical Scenario)

This document describes a complete SSH hardening scenario performed in a virtualized lab environment as part of LPIC-2 practical exercises.  
The focus of this lab is not only on making SSH functional, but on understanding authentication flow, access control, logging behavior, and basic intrusion mitigation on a Linux system.

Environment:
- Server: Ubuntu (systemd-based)
- Clients: Fedora (primary)
- Platform: Virtual Machines (isolated lab environment)

---

## 1. Problem Definition

The main goal of this scenario was to design a secure SSH configuration with the following objectives:

- Enable and manage SSH using systemd
- Enforce public key authentication
- Completely disable password-based SSH login
- Disable root login over SSH
- Restrict SSH access to a specific user
- Observe and analyze authentication behavior via logs
- Automatically block repeated failed login attempts
- Understand the interaction between SSH, PAM, and Fail2Ban

---

## 2. Implementation Steps

### 2.1 SSH Service Setup

The SSH service was enabled and started on the Ubuntu server:

systemctl enable ssh  
systemctl start ssh  
systemctl status ssh  

Listening ports were verified:

ss -tulnp | grep ssh  

---

### 2.2 SSH Key-Based Authentication

An SSH key pair was generated on the client system (Fedora):

ssh-keygen -t ed25519  

(Initial mistake: the key pair was first generated on the server itself.  
Correction: SSH keys must always be generated on the client side, since the client proves its identity to the server.)

The public key was copied to the server:

ssh-copy-id user@server_ip  

Verification on the server:

cat /home/user/.ssh/authorized_keys  

Permissions were checked to ensure SSH would accept the key:

ls -ld /home/user/.ssh  
ls -l /home/user/.ssh/authorized_keys  

---

### 2.3 SSH Host Keys Clarification

While inspecting SSH-related files, the following file was examined:

/etc/ssh/ssh_host_ed25519_key  

(Initial confusion: this file was mistaken for a client authentication key.  
Correction: files under `/etc/ssh/ssh_host_*` are server host keys, used to identify the server to clients.  
Client authentication keys are stored in `/home/user/.ssh/authorized_keys`.)

---

### 2.4 SSH Daemon Hardening

The SSH daemon configuration file was edited:

/etc/ssh/sshd_config  

The following security-related changes were applied:

PermitRootLogin no  
PasswordAuthentication no  
AllowUsers user  

The SSH service was restarted to apply changes:

systemctl restart ssh  

Results:
- Root login over SSH was blocked
- Password-based authentication was rejected
- Only the specified user could authenticate using a public key

---

## 3. Authentication and Log Analysis

Authentication behavior was verified using system logs instead of assumptions.

Primary log file:

/var/log/auth.log  

Observed entries included:
- pam_unix session open and session close messages
- Successful authentication entries such as:
  Accepted publickey
- Failed authentication attempts
- Denied root login attempts
- Source IP addresses of SSH clients

This confirmed that:
- SSH restrictions were working as intended
- Authentication flow was traceable
- PAM handled session lifecycle correctly

---

## 4. Fail2Ban Configuration and Testing

Fail2Ban was installed and enabled:

systemctl enable fail2ban  
systemctl start fail2ban  

Jail status was checked:

fail2ban-client status  
fail2ban-client status sshd  

Fail2Ban logs were reviewed:

/var/log/fail2ban.log  

Observed behavior:
- Repeated failed SSH login attempts were detected
- Source IP addresses were automatically banned
- Ban events were logged and traceable

---

## 5. Security Considerations

Fail2Ban improves SSH security but does not provide complete protection on its own.  
For a more robust setup, it should be combined with additional access control mechanisms such as:

- Firewall rules (iptables, nftables, firewalld)
- Network-level access restrictions
- SSH configuration hardening
- Host-based access control using:
  /etc/hosts.allow  
  /etc/hosts.deny  

These mechanisms together form a layered security approach.

---

## 6. Evidence and Documentation

The following logs can be used as evidence for this scenario:

- /var/log/auth.log
  - Successful public key authentication
  - Failed login attempts
  - Root login denial
  - PAM session lifecycle events

- /var/log/fail2ban.log
  - Detection of failed authentication attempts
  - IP ban events
  - SSH jail activity

Screenshots of these logs can be attached to demonstrate correct system behavior.


### Log Analysis with journalctl

As part of this scenario, system logs were analyzed using `journalctl` to verify SSH behavior and authentication flow.

The following aspects were observed and validated through logs:

- SSH service activity and status using:
  journalctl -u ssh
  journalctl -u sshd

- Successful and failed authentication attempts, including:
  - Accepted public key authentication
  - Failed password attempts
  - Attempts to log in as unauthorized users (e.g., root)

- PAM (Pluggable Authentication Modules) interactions, where logs indicated:
  - Session opening and closing events
  - Authentication success or failure
  - User identity and UID involved in each session

- Source IP addresses and ports used during SSH connection attempts, allowing identification of:
  - Legitimate client connections
  - Repeated failed attempts from the same IP (later correlated with Fail2Ban actions)

Example commands used during analysis:
- journalctl -u ssh
- journalctl -u ssh -f
- journalctl | grep ssh
- journalctl | grep pam_unix

These logs were used to confirm that:
- Password-based authentication was successfully disabled
- Access was restricted to explicitly allowed users
- Key-based authentication was functioning as expected
- Security mechanisms such as Fail2Ban were triggered based on log patterns


This log-based validation ensures that configuration changes are not only applied but also verifiable at runtime through system-level auditing  
ت (smiley face)