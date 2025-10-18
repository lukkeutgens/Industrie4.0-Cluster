# Securing SSH Access
Working via terminal over OpenSSH is of course more convenient than using the physical console, but it also requires securing the OpenSSH service.  
To do this, we will set up public/private key authentication.  
I use [PuTTY](https://putty.org/index.html) and `PuTTYgen` for this, as they are lightweight and simple tools.  
For Linux/macOS, the tool `ssh-keygen` is more commonly used — but I'm working on a Windows 11 

Some links with more info:
- https://0ut3r.space/2023/10/24/openssh-hardening/
- https://documentation.suse.com/sles/15-SP6/html/SLES-all/cha-ssh.html
- https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement

---

## Generate SSH Client keys
1. Open `PuTTYgen`
2. Select `EdDSA` and choose `Ed25519 (255 bits)`  
3. Click `Generate`  
4. While generating move your mouse randomly to generate entropy  
5. Add a `Key passphrase` and confirm it
6. Copy the **OpenSSH-format public key** (shown in the top box) into a `.txt` file for later use
7. Click `Save public key`
8. Click `Save private key`

> ⚠️ Note 1:  
> The saved public key (step 7) is in SSH2 format, which is not compatible with OpenSSH servers.  
> That’s why we also copy-paste the OpenSSH-format key (e.g. `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB... user@WindowsPC`) into a separate `mykey.txt` file.

> ⚠️ Note 2:
> The private key should be readable only by the owner, you will have to change the permissions for that file on the system where it is stored.
---
## Copy Public Key to Server (Manual Method)
After generating your key in PuTTYgen, you should have copy-pasted the OpenSSH-format public key into a .txt file. Now we’ll transfer that file to the server and append the key manually.
Use scp to transfer the file to your home directory on the server:
```bash
scp -P 22348 mykey.txt luk@192.168.50.160:~
```
- -P 22348 : Use your custom SSH port
- luk@192.168.50.160 : Replace with your actual username and server IP
- :~ : Places the file in the user's home directory

SSH into the server and append the key:
```bash
cat ~/mykey.txt >> ~/.ssh/authorized_keys
```
Then set correct permissions:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
- chmod 700 : Owner may read (r), write (w) and execute (x). For directories, x means the owner can enter and access files inside. Without it, the directory is unusable.
- chmod 600 : Owner can read (r) and write (w). Group and others have no access — ideal for sensitive files like authorized_keys.
⚠️ If these permissions are too open (e.g. 755 or 644 on private files), SSH will refuse to use the key and may show errors like: `Bad owner or permissions on ~/.ssh/authorized_keys`

Remove the temporary file:
```bash
rm ~/mykey.txt
```

---

## Create a legal notice
First, create a legal notice that will be displayed as a banner when someone attempts to log in (optional):
```bash
vi /etc/issue.net
```
Add content:
```bash
Authorized access only!
All activities are logged.
Unauthorized access may result in penalties under applicable law.
```
Save and exit the file.

---

## OpenSSH Server Configuration
Edit the OpenSSH server config-file:
```bash
sudo vi /etc/ssh/sshd_config
```
Adjust the parameters:
```bash
Port 22348                            # Change default port to reduce automated scans

# Logging
LogLevel VERBOSE                      # Enables detailed logging (e.g. username, key fingerprint)

# Authentication
LoginGraceTime 60s                    # Allow only 60 seconds to complete login
PermitRootLogin no                    # Disable direct root login
StrictModes yes                       # Check file permissions and ownership of user files
MaxAuthTries 3                        # Limit to 3 authentication attempts per connection
MaxSessions 10                        # Limit to 10 concurrent sessions per connection
PubkeyAuthentication yes              # Enable public/private key login
ChallengeResponseAuthentication no    # Legacy option, usually disabled unless required by specific PAM modules
#Access control
#AllowGroups ipausers admins          # Restrict access to specific groups (commented out)
AllowUsers luk                        # Allow only specific users (comment out if using AllowGroups)
AuthorizedKeysFile	%h/.ssh/authorized_keys  # Path for user's public key file

# Passwords
PasswordAuthentication no              # Disable password login (key-only access)
KbdInteractiveAuthentication no        # Disable keyboard-interactive (used by PAM for passwords)

# PAM integration
UsePAM yes                             # Enable PAM (required for TOTP, audit, etc.)
#AuthenticationMethods publickey, keyboard-interactive  # Uncomment when PAM with TOTP is configured

Banner /etc/issue.net                  # Display legal notice at login
```

---

## Validate SSH Configuration
After making changes to the SSH configuration file, always check for syntax errors before restarting the service:
```bash
sudo sshd -t
```
If no output is returned, the syntax is valid. If there are errors, they will be displayed in the terminal.

Restart the SSH service:
```bash
sudo systemctl restart sshd
```
Check active configuration values:
```bash
sudo sshd -T | grep -Ei 'port|auth|banner|pam'
```

---

## Check login protection
Try logging in via IP without using SSH keys — this should be denied. 
Then check the logs:
```bash
sudo journalctl -u ssh
sudo journalctl -u ssh --no-pager -n 50
```

