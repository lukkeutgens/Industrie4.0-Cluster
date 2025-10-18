# OS-Hardening
⚠️ **NOTE** : This project is a learning project. This is not a guide to full os-hardening. It's the first steps to learn about it.

I have installed 4 debian servers and did already some post-installation configurations. In this step I will add some first OS-Hardening steps to make the servers more secure.
It's not my plan to make them ISO compliant or production save as this is a learning project. But then, it's better to start with security in mind, so first close everything and then open only whats needed.

## Steps taken
1. Configure updates and package management
2. Install postfix to send alert emails
3. Disable unnecessary services
4. Set timezone on servers (certificate handling)
5. Clean up unused packages
6. Manage users (no root login, use of sudo)
7. Configure SSH with public key authentication (no passwords)
8. Install and configure firewall
9. Add intrusion detection (fail2ban)
10. Add system auditing (auditd)
11. Configure AppArmor profiles

## Steps to do
1. Define and implement backup strategy
2. ...

