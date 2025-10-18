# Disable Unused Services
To reduce the attack surface and prevent unnecessary ports from being exposed, we disable services that are not needed and may be active by default.

### Services to disable:
- **avahi-daemon**: Netwerkservice-discovery via mDNS (like Bonjour) - Not needed on servers without plug-and-play devices such as printers or smart TVs. May open unnecessary ports and generate unwanted traffic.
- **cups**: Print server (Common Unix Printing System) - Only useful if you want to print locally. On headless servers, it's redundant and introduces a potential attack surface.
- **bluetooth**: Bluetooth stack for wireless devices - Irrelevant on servers without a GUI or physical Bluetooth hardware. Disabling it saves resources and reduces the attack vector.

## Check if a service is installed:
Replace `servicename` with the actual name:
```bash
dpkg -l | grep servicename
```

## Disable and Mask Services:
Run the following commands for each service you want to disable:
```bash
systemctl disable bluetooth	
systemctl mask bluetooth
systemctl status bluetooth
```

Note: On our servers, avahi-daemon and cups are not installed because we use a headless setup without GUI.
