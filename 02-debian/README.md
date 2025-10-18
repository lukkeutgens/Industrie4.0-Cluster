# Debian Setup
This folder documents the post-installation steps I have taken for the Debian 12 headless servers.

## Why Debian?
- **Stability**: Proven reliability
- **Modularity**: Clean base system, ideal for custom infrastructure
- **Security**: Strong community support and predictable update cycles
- **Transparency**: No vendor lock-in, fully open-source
- **Learning value**: Forces deeper understanding of Linux internals and manual configuration

---

## Quick install steps
1. Select Install (not the graphical installer).
2. Choose language: Dutch, and location: Belgium.
3. Set the hostname for your server.
4. Set the domain name: iot.keutgens.be.
5. Define the root password.
6. Create a user account with password.
7. Choose manual disk partitioning if needed.
8. Configure your volumes as needed.
9. When asked about additional installation media, choose No.
10. Select a Belgian mirror for package downloads.
11. Leave the proxy field empty.
12. Participation in the package usage survey is optional — choose freely.
13. When prompted to select additional software, choose only:
    - SSH server
    - Standard system utilities
14. Install the GRUB bootloader to /dev/sda.
15. At the end, Debian will ask if the installation media has been removed.
      - In Oracle VirtualBox, go to Devices → Optical Drives and ensure the Debian ISO is unchecked (this usually happens automatically).
      - Then choose Continue.
16. The system will reboot and start up. Installation is complete.

---

## Post-Install Configuration Steps
Each server was configured with the following steps:

- **VIM Basic install**  
  Debian ships with `vim.tiny` by default, which is too limited for practical use. Installed full `vim` package.

- **Fixed IP address**  
  Configured static IP via `/etc/network/interfaces` for predictable node communication.

- **Hostname & Domain**  
  Set hostname and local domain (`iot.keutgens.be`) for LAN-based DNS resolution.

- **Welcome screen (MOTD)**  
  Created a custom Message of the Day for a personal touch during SSH login.

- **OS Hardening**  
  Applied basic security measures:
  - Disabled root SSH login
  - Configured UFW firewall
  - Removed unused packages
  - Created non-root user with sudo access
 
  > This is not intended to be ISO-compliant or production-grade — it's a learning project.

---

## Folder Contents
```plaintext
02-debian/
├── docs
│   ├── 01-vimbasic.md                   # Installing VIM-Basic
│   ├── 02-networksetup.md               # Setup fixed IP-Addresses & Domain names
│   ├── 03-loginmessage.md               # Show login message (MOTD)
│   └── os-hardening
│       ├── 00-README.md                 # Description steps OS-hardening
│       ├── 01-unattended-updates.md     # Configure unattended updates
│       ├── 02-postfix.md                # Install postfix email service
│       ├── 03-disable-services.md       # Disable unnecessary services
│       ├── 04-timezone.md               # Set timezone on servers
│       ├── 05-cleanup.md                # Clean up unused packages
│       ├── 06-useraccess.md             # Manage users
│       ├── 07-secure-ssh.md             # Configure SSH with public key
│       ├── 08-firewall.md               # Install and configure firewall
│       ├── 09-fail2ban.md               # Add intrusion detection
│       ├── 10-auditd.md                 # Add system auditing
│       └── 11-apparmor.md               # Configure AppArmor profiles
├── files
│   ├── 99-custom                        # For welcome message (MOTD)
│   ├── aliases                          # Used with postfix
│   ├── apparmor.d
│   │   ├── postfix-smtp                 # Postfix profile
│   │   └── usr.sbin.auditd              # Auditd profile
│   ├── audit
│   │   ├── auditd.conf                  # Auditd configuration file
│   │   └── rules.d
│   │       └── csrv1.rules              # Server rules
│   ├── chrony.conf                      # Timezone setup with chrony
│   ├── enp0s3.cfg                       # Network interface
│   ├── fail2ban
│   │   ├── jail.d
│   │   │   └── recidive.local           # Recidive jail file
│   │   └── jail.local                   # Jail configuration file
│   ├── hosts                            # Network hosts file
│   ├── issue.net                        # Legal message when logged in
│   ├── journald.conf                    # Logging config
│   ├── postfix
│   │   ├── main.cf                      # Postfix main configuration
│   │   └── sasl                         # Gmail sasl password
│   ├── server-description               # Used in welcome message
│   ├── sshd_config                      # sshd server configuration
│   └── vimrc                            # VIM Config (colorscheme change)
└── README.md                            # This file

```


