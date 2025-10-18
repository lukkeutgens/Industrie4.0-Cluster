# Application Confinement with AppArmor
AppArmor is a Linux security module that restricts what applications can do, even when running as root.  
It enforces per-application profiles that define allowed file access, capabilities, and system interactions.

## AppArmor on Debian 12
AppArmor is installed by default on Debian 12, but additional setup is required per server.

### Required Packages
```bash
sudo apt install rsyslog
sudo apt install apparmor-utils
sudo apt install apparmor-profiles apparmor-profiles-extra
```
Ensure rsyslog is running:
```bash
sudo systemctl status rsyslog
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
```
Verify that /var/log/syslog exists and is owned by root:
```bash
ls -l /var/log/syslog
```
Enable forwarding from journald to rsyslog:
```bash
sudo vi /etc/systemd/journald.conf
```
Uncomment or add:
```bash
ForwardToSyslog=yes
```

---

## AppArmor Status and Modes
Check active profiles:
```bash
sudo aa-status
```
AppArmor modes:
- `Unconfined` : Profile exists but is not enforced
- `Complain` : Violations are logged but not blocked
- `Enforce` : Violations are blocked (service may crash or stop)

Change mode:
```bash
sudo aa-complain /etc/apparmor.d/<profile>
sudo aa-enforce /etc/apparmor.d/<profile>
sudo aa-disable /etc/apparmor.d/<profile>
```
Restart the affected service:
```bash
sudo systemctl restart <service>
```
View logs:
```bash
journalctl -u apparmor | grep <profile>
sudo aa-logprof
```

---

## Manage AppArmor Profiles
Add, replace, or remove profiles:
```bash
sudo apparmor_parser -a /etc/apparmor.d/<profile>   # Add
sudo apparmor_parser -r /etc/apparmor.d/<profile>   # Replace
sudo apparmor_parser -R /etc/apparmor.d/<profile>   # Remove
```

Generate a new profile:
```bash
sudo aa-genprof <application>
```
In a second terminal, start or restart the target service. Use S to scan and F to finish. Do this a couple off times until ok.
> ⚠️ Note: If multiple log files are detected, edit the profile manually and replace /log1, /log2, etc. with /logs/**.

---

## Securing SSHD with AppArmor
Search in the directory `/usr/share/apparmor/extra-profiles` for the `sshd` profile and then add it to AppArmor.
```bash
cd /usr/share/apparmor/extra-profiles/                 # Extra profiles
ls -l | grep sshd                                      # Search for sshd
sudo cp usr.sbin.sshd /etc/apparmor.d/                 # Copy profile to AppArmor
sudo apparmor_parser -a /etc/apparmor.d/usr.sbin.sshd  # Add the profile in AppArmor 
sudo aa-complain /etc/apparmor.d/usr.sbin.sshd         # Set profile to complain mode
sudo aa-status | grep sshd                             # Check profile errors in complain mode
sudo systemctl restart sshd                            # Restart sshd profile
sudo aa-logprof                                        # Check for errors
```
If no errors are found:
```bash
sudo aa-enforce /etc/apparmor.d/usr.sbin.sshd          # Set profile to enforce
```

---

## Securing Postfix with AppArmor
Search in the directory `/usr/share/apparmor/extra-profiles` for the `profix` profile and then add it to AppArmor.
```bash
cd /usr/share/apparmor/extra-profiles/
ls -l | grep postfix                           # There are many
sudo cp postfix-* /etc/apparmor.d/             # Copy them all
sudo systemctl restart apparmor                # Restart AppArmor will activate them
sudo aa-status | grep postfix                  # Check if they are active
```
I found an error in the postfix-smtp profile:
```bash
vi /etc/apparmor.d/postfix-smtp
```
Add line at the bottom:
```bash
/etc/postfix/sasl/** rk,
```
Reload profile:
```bash
sudo apparmor_parser -r /etc/apparmor.d/postfix-smtp
sudo systemctl restart apparmor
```
Test email:
```bash
echo "Test local mail" | mail -s "Test" ********@gmail.com
```
Check mail logs:
```bash
sudo tail -n 50 /var/log/mail.log
```

---

## Securing auditd with AppArmor
Auditd is notoriously difficult to confine with AppArmor due to its low-level kernel interactions and extensive logging behavior.  
We start by generating a profile in `complain mode`, which logs violations without enforcing restrictions.

### Profile Generation Workflow
In **Terminal 1**, start profile generation:
```bash
sudo aa-genprof auditd
```
In Terminal 2, restart the auditd service (it may crash during profiling):
```bash
sudo systemctl restart auditd
```
Back in Terminal 1:
- Press S to scan
- Press F to finish

Then switch auditd to complain mode:
```bash
sudo aa-complain /etc/apparmor.d/usr.sbin.auditd
sudo aa-status | grep auditd
```
If auditd runs without crashing:
```bash
sudo aa-logprof
```

### Fixing Log Path Issues
There will be errors as `log1`, `log2`, ... Then stop the scan (F) and manually edit the profile to generalize log paths. Search for `log1` en adjust to `/log/**`.
```bash
sudo vi /etc/apparmor.d/usr.sbin.auditd
```
Add or replace:
```bash
owner /var/log/audit/** rw,
```

### Sample profile:
```bash
vi usr.sbin.auditd
```
Content:
```bash
# Last Modified: Sat Jul 26 15:56:29 2025
abi <abi/3.0>,

include <tunables/global>

/usr/sbin/auditd flags=(complain) {
  include <abstractions/base>
  include <abstractions/gstreamer>

  /usr/sbin/auditd mr,
  owner /etc/passwd r,
  owner /var/log/audit/** rw,

}
```
