# System Auditing with auditd
auditd is a powerful auditing daemon that logs detailed security-related system activity.  
It provides deeper insight than traditional logging tools like `syslog`, capturing kernel-level events and system calls.

https://manpages.debian.org/testing/auditd/auditd.8.en.html

## Why Use auditd?
- Tracks **security-relevant actions** such as:
  - Who modified `/etc/passwd`
  - Which commands were executed with `sudo`
- Essential for:
  - Compliance (e.g. ISO 27001, NIST, GDPR)
  - Forensic analysis after incidents
  - Authentication auditing and privilege tracking
- Operates at a lower level than syslog — directly from the Linux kernel via the Netlink interface

⚠️ auditd is especially valuable on hardened systems, servers with sensitive data, or environments requiring traceability.

---

## Installation
```bash
sudo apt update
sudo apt install auditd audispd-plugins
```
Edit the configuration file:
```bash
sudo vi /etc/audit/auditd.conf
```
Recommended parameters:
```bash
log_file = /var/log/audit/audit.log
log_format = RAW
max_log_file = 8
num_logs = 5
space_left_action = SYSLOG
admin_space_left_action = SUSPEND
name_format = HOSTNAME
```

---

## Create Audit Rules
Now we need to define the audit rules that should be enforced:
- Track SSH logins and session activity
- Monitor user management actions (e.g. new users, sudo, password changes)
- Record command usage
- Log access to sensitive files

Create the rules file:
```bash
vi /etc/audit/rules.d/csrv1.rules
```
Add content:
```bash
# Monitor changes to user files
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity

# Monitor on sudo use
-w /usr/bin/sudo -p x -k sudo
-w /etc/sudoers -p wa -k sudo

# Monitor time settings
-a always,exit -F arch=b64 -S adjtimex -S settimeofday -S clock_settime -k time-change

# Monitor kernel module changes
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules

# Monitor auditd itself (protection against sabotage)
-w /etc/audit/ -p wa -k auditconfig
-w /var/log/audit/ -p wa -k auditlogs

# Monitor SSH daemon socket
-w /var/run/sshd -p rwxa -k sshd_socket

# Monitor session creation (ex. SSH login)
-a always,exit -F arch=b64 -S pam_start -k ssh_session
-a always,exit -F arch=b64 -S session -k ssh_session

# Monitor commands execution through SSH(execpt for root)
-a always,exit -F arch=b64 -F euid=0 -S execve -k ssh_exec

# Monitor changes to the SSH configuration
-w /etc/ssh/sshd_config -p wa -k ssh_config
```
Save and exit the file.
Restart auditd:
```bash
systemctl restart auditd
systemctl status auditd
```

## Verify auditd is active
```bash
sudo auditctl -l                  # List active audit rules
sudo ausearch -k sshd_activity    # View SSH activity (after login)
sudo ausearch -k command_exec     # Show executed commands
sudo ausearch -k user_change      # Show user modifications
sudo ausearch -k sudo_log         # Show sudo usage
sudo ausearch -m USER_LOGIN -I    # Show all user login events
sudo ausearch -ua luk -m USER_LOGIN -ts today -I  # Show today's logins for user 'luk'
```
> ⚠️ Note: Failed SSH login attempts can be retrieved via `journalctl -u ssh.service`

