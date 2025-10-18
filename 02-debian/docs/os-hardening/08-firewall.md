# 🔥 Firewall Configuration on Debian

Debian comes with a built-in firewall engine — the **Netfilter** module in the Linux kernel.  
However, it does **not include a user-friendly frontend** by default.  
To simplify firewall management, we install **UFW (Uncomplicated Firewall)**, which integrates well with Debian.

---

## 🛠️ Install and Configure UFW

```bash
sudo apt install ufw                      # Install UFW
sudo ufw default deny incoming            # Block all incoming traffic by default
sudo ufw default allow outgoing           # Allow all outgoing traffic
sudo ufw allow 22348/tcp                  # Allow incoming SSH on custom port
sudo ufw enable                           # Enable the firewall
```
Check status:
```bash
sudo ufw status verbose
```

## How Firewalls Work in Linux
- The Netfilter module in the Linux kernel is the actual firewall engine.
- Tools like iptables, nftables, ufw, and firewalld are just configuration interfaces.
- When you define rules using one of these tools, they are immediately loaded into the kernel.
- The kernel enforces these rules as soon as network traffic arrives — no background service is needed.

UFW is a frontend for iptables, making rule management easier for most use cases.

