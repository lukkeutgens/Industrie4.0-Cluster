## Unattended Updates
Unattended updates are essential in OS hardening because they ensure that critical security patches are applied automatically, without manual intervention. 
This reduces the window of vulnerability and protects the system against known exploits — especially in environments where manual updates may be delayed or overlooked.

To enable automatic updates, we install two additional packages:
- `unattended-upgrades`: Automatically installs the latest security updates
- `apt-listchanges`: Provides insight into what changed during upgrades

More info:
- [Debian Wiki: Unattended Upgrades](https://wiki.debian.org/UnattendedUpgrades)
- [std.rocks: Debian Auto Update Guide](https://std.rocks/gnulinux_debian_auto_update.html)

---

## First update the system
```bash
apt update
apt full-upgrade -y
```

---

## Install Required Packages
```bash
apt install unattended-upgrades 
apt install apt-listchanges
```

---

## Configure Unattended Upgrades
Adjusting the configuration file is optional and not always necessary. However, we do want to receive emails when updates have been applied.
```bash
sudo vi /usr/share/unattended-upgrades/50unattended-upgrades
```
To receive email notifications when updates are applied, add:
```bash
Unattended-Upgrade::Mail "luk.keutgens@gmail.com";
Unattended-Upgrade::MailReport "on-change";
```

---

## Check Systemd Timers
Verify that the timers "apt-daily.timer" and "apt-daily-upgrade.timer" are active:
```bash
systemctl list-timers –-all
```
If they’re missing, enable them:
```bash
systemctl enable –-now apt-daily.timer apt-daily-upgrade.timer
```

## Test the Setup
Run a dry test to verify functionality:
```bash
Unattended-upgrade –-dry-run –-debug
```

Now the system will keep it self more secure by automatically updating the latest security patches
