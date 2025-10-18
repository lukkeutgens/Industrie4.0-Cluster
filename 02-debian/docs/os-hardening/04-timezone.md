# Setup correct timezone
The date and time on our servers must always remain accurate to ensure proper logging and to support other software packages such as fail2ban, auditd, and Grafana, which also rely on correct timestamps.

This can be done using:
- The default `timedatectl` tool  
- Or by using `Chronyd` as an alternative time synchronization service

We will use `Chronyd`, but include `timedatectl` for learning purposes.

---

## Setup with timedatectl
Check current settings:
```bash
timedatectl
```
Expected output:
- Local time:    Sat 2025-07-19 10:46:00 CEST
- Universal time: Sat 2025-07-19 08:46:00 UTC
- RTC time:      Sat 2025-07-19 08:46:00
- Time zone:     Europe/Brussels (CEST, +0200)
- System clock synchronized: yes
- NTP service:   active
- RTC in local TZ: no

Check if systemd-timesyncd is installed and enabled. If not, install and activate it:
```bash
dpkg -l | grep systemd-timesyncd      # Check if installed
systemctl start systemd-timesyncd     # Start service
systemctl enable systemd-timesyncd    # Enable at boot
systemctl status systemd-timesyncd    # Check status
timedatectl set-ntp true              # Activate NTP
timedatectl                           # Verify again
```
These steps will synchronize the server’s time. However, Chronyd is more accurate and preferred for production use.

---

## Setup with Chronyd
Install Chrony:
```bash
sudo apt update
sudo apt install chrony
sudo systemctl status chrony
```
Edit the configuration file:
```bash
vi /etc/chrony/chrony.conf
```
Recommended content:
```bash
# Use debian vendor zone.
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
server 0.be.pool.ntp.org iburst
server 1.be.pool.ntp.org iburst
```
- `pool`: Dynamic server groups
- `iburst`: Speeds up initial synchronization
- `server`: Static fallback servers

Save and exit the file, then restart Chrony:
```bash
systemctl restart chrony
systemctl status chrony
```
Verify Chrony status:
```bash
chronyc sources -v     # Shows state of NTP sources
chronyc tracking       # Shows sync status (Leap = Normal means OK)
```
- sources -v : Gives the state off the sources
- tracking : Leap = Normal (Ok)

With Chrony active, your server will maintain accurate time — essential for logging, monitoring, and security tools.

