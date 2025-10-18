# Postfix (Email) install
With Postfix it will be possible to send emails when there is a problem. For example when fail2ban detects an attack and bans those systems.

---

## Install postfix
```bash
sudo apt update
sudo apt install postfix mailutils
```
During installation, you'll get a configuration screen. I'm using Gmail as a smarthost:
- Email configuration type: Internet with smarthost
- System mail name: servername.iot.keutgens.be
- SMTP relay server: [smtp.gmail.com]:587

---

## After installing postfix
Edit the aliases file:
```bash
sudo vi /etc/aliases
```
Adjust the content:
```bash
# See man 5 aliases for format
postmaster: 		root
root: luk.keutgens@gmail.com
```
Save and exit the file, then apply the changes:
```bash
sudo newaliases
```

---

## Adjust Postfix Configuration
During installation the config-file was made, but we will need to make some changes.
Edit the main configuration file:
```bash
vi /etc/postfix/main.cf
```
Add or adjust the following lines:
```bash
smtp_tls_security_level=encrypt
myorigin = csrv1.iot.keutgens.be

inet_protocols = ipv4

# Gmail authenticatie-instellingen
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```
Explanation:
- smtp_tls_security_level = encrypt: Enforces encryption (instead of optional)
- myorigin: Can also be set via /etc/mailname, but direct assignment is valid
- inet_protocols = ipv4: Use all if you want IPv6 support

---

## Setup Gmail App Password
To use Gmail, we need to create an App Password via https://myaccount.google.com/apppasswords
We will store our password in the file /etc/postfix/sasl/sasl_passwd.

### ⚠️ Warning!
Google may return the password with spaces — remove all spaces before using it!

Edit the password file:
```bash
vi /etc/postfix/sasl/sasl_passwd
```
Add the content:
```bash
[smtp.gmail.com]:587 luk.keutgens@gmail.com:mypasswordwithoutspaces
```
Save and exit the file.
Load the file:
```bash
sudo postmap /etc/postfix/sasl/sasl_passwd
```
Restart postfix service:
```bash
systemctl restart postfix
systemctl status postfix
```

---

## Send an email
Let's test we can send an email. Try out the following command:
```bash
echo "This is a test email from Postfix on csrv1." | mail -s "Postfix Testmail" luk.keutgens@gmail.com
```

---

## Check Logs and Mail Queue
Use journalctl to inspect Postfix logs:
```bash
journalctl -u postfix                        # Check postfix service
journalctl | grep '********@gmail.com'       # Returns everything filtered on email address
journalctl -n 100                            # Gives back the final 100 messages
```

Check mail queue:
sudo postqueue -optie
- -p : Print the full mail queue
- -f : Force an immediate flush of the queue
- -i queue_id : Limit the action to the specified queue ID (for a single message)
- -j : Output the queue in JSON format
- -s : Show only a summary of the queue

For example return the full mailqueue:
```bash
sudo postqueue -p
```

