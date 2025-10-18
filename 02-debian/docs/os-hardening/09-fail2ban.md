# Login protection with fail2ban
Fail2ban is a security tool that protects your server against brute-force attacks by monitoring log files and blocking suspicious IP addresses.

### More Info
- [Fail2ban GitHub Repository](https://github.com/fail2ban/fail2ban)  
- [Secure Debian 12 with Fail2ban and nftables (Pieter Bakker)](https://pieterbakker.com/secure-debian-12-with-fail2ban-and-nftables/)  
- [Fail2ban GitHub Discussions](https://github.com/fail2ban/fail2ban/discussions/3638)

## Why use Fail2ban?
- Blocks IP addresses after repeated failed login attempts
- Works by analyzing system logs (e.g. journalctl, /var/log/auth.log)
- Especially effective against brute-force attacks on SSH, mail servers, and other exposed services
- Uses configurable jails, rules that define how many attempts are allowed and what action to take
- Prevents attackers from trying thousands of password combinations

⚠️ Without Fail2ban, a public-facing SSH port is vulnerable to automated bots and brute-force scanners.

---

## Fail2ban installation
Installation:
```bash
sudo apt update
sudo apt install fail2ban
```
We create our own `jail.local` file to override default settings from `jail.conf`. This ensures our configuration is preserved during package updates.

Create the local file:
```bash
sudo vi /etc/fail2ban/jail.local
```

Add content:
```bash
[DEFAULT]
# Logging and database
logtarget = /var/log/fail2ban.log
dbpurgeage = 1d
dbfile = /var/lib/fail2ban/fail2ban.sqlite3

# Send email when banned
sender = csrv1@iot.keutgens.be
sendername = CSRV1 Fail2Ban Alert
destemail = *********@gmail.com
action = %(action_mw)s
mta = sendmail

# UFW Firewall actions
banaction = ufw
banaction_allports = ufw

[sshd]
enabled  = true
port     = 22348
filter   = sshd
backend  = systemd
logpath  = _SYSTEMD_UNIT=ssh.service
maxretry = 3
bantime  = 1h
findtime = 10m
```

Some extra information for sending the email command in the config file:
- `action = %(action_mwl)s` : Ban, log, and send full email with logs
- `action = %(action_mw)s` : Ban and send email with IP and hostname
- `action = %(action_m)s` : Ban and send minimal email
- `action = %(action_)s` : Ban without sending email

---


## Fail2ban Recidive Jail
The recidive jail is designed to catch repeat offenders, IP addresses that were previously banned by other jails and continue to attack. 
It acts as a second layer of defense, applying longer bans to persistent threats.

> ⚠️ Note:
> The `jail.local` file is placed directly in the main `/etc/fail2ban/` directory because it overrides the default `jail.conf` configuration.  
> In contrast, the `recidive.local` file goes into the `/etc/fail2ban/jail.d/` folder, as it defines a modular, additional jail that complements the core setup.

Create the file:
```bash
sudo vi /etc/fail2ban/jail.d/recidive.local
```
Add content:
```bash
[recidive]
enabled = true
logpath = /var/log/fail2ban.log
banaction = ufw
backend = systemd
maxretry = 5
findtime = 1d
bantime = 1w
```
Explanation:
- `enabled true` : Activates the jail
- `logpath /var/log/fail2ban.log` : Monitors Fail2ban's own log for repeated bans
- `banaction ufw`	: Uses UFW to block offending IPs
- `backend systemd`	: Uses systemd journal for log parsing (recommended for Debian 12)
- `maxretry 5` : Triggers if an IP was banned 5 times within the findtime window
- `findtime 1d` : Looks back over 1 day to count retries
- `bantime 1w` : Bans the IP for 1 week

Check status
```bash
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
sudo fail2ban-client status
```

---

## Troubleshooting Fail2ban
If Fail2ban is not working, restart it and view detailed logs:
```bash
systemctl stop fail2ban
fail2ban-server -xf start
```

Check current bans:
```bash
sudo fail2ban-client status              # Get state bans and jails
sudo fail2ban-client status sshd         # Shows more details
sudo fail2ban-client unban –all          # Unban all
sudo fail2ban-client unban <IPaddress>   # Unban a certain IP
```

To reset the SSH jail:
```bash
sudo fail2ban-client stop sshd
sudo fail2ban-client start sshd
```
