# Network Setup: Fixed IP & Domain
This file documents how I configured a **static IP address** and set the **hostname and domain** on each Debian 12 node in my Industry 4.0 cluster.

---

## Configure Static IP
For predictable communication between nodes, especially in a Kubernetes environment, each server needs a fixed IP. This avoids DHCP-related issues and ensures that services like Longhorn, MetalLB, and Ingress can reliably route traffic.

De servers will get the following IP-adressess:
-	csrv1: 192.168.50.160 (control plane server)
-	wsrv1: 192.168.50.161 (worker node 1 server) 
-	wsrv2: 192.168.50.162 (worker node 2 server) 
-	wsrv3: 192.168.50.163 (worker node 3 server) 

Edit the network interfaces file and adjust IP for each server as described:
```bash
sudo vi /etc/network/interfaces.d/enp0s3.cfg

auto enp0s3
iface enp0s3 inet static
	address 192.168.50.160
	netmask 255.255.255.0
	gateway 192.168.50.1
	dns-nameservers 192.168.50.1 8.8.8.8
```
Save and exit the file, then restart the networking service:
```bash
systemctl restart networking
```
Verify the IP address with:
```bash
ip a
```

---

## Setup hostname & domain
- Ensures consistent DNS resolution across the cluster
- Simplifies service discovery and TLS setup
- Avoids issues with DHCP lease changes
- Aligns with LAN DNS (iot.keutgens.be) and internal routing

Set server hostname, repeat for each server
```bash
sudo hostnamectl set-hostname csrv1
```
Note that we don't add the domain name after it, this is for more flexibility.

Setup de hosts file so each server can find the other
```bash
sudo vi /etc/hosts

192.168.50.160	csrv1.iot.keutgens.be	csrv1
192.168.50.161	wsrv1.iot.keutgens.be	wsrv1
192.168.50.162	wsrv2.iot.keutgens.be	wsrv2
192.168.50.163	wsrv3.iot.keutgens.be	wsrv3
```
Check the setup with the following command's
```bash
hostname		# Shows the server name
hostname -d		# Shows only the domain name
hostname -f		# Shows server name with domain (full server name)
```




