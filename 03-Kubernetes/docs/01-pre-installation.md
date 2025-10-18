# Pre-Installation
Before installing Kubernetes, we need to prepare the servers. This chapter describes all the steps I’ve taken.
---

## 1. Set server names
Each server (node) must have a unique hostname for Kubernetes. This was already configured during the post-installation of Debian. See those docs for details.

To verify the full system UUID on each server:
```bash
sudo dmidecode -s system-uuid
```

---

## 2. Open needed network ports
Kubernetes Port Reference: https://kubernetes.io/docs/reference/networking/ports-and-protocols/

Open the ports listed below on all nodes. Note the difference between control plane nodes and worker nodes.

For the control plane node:
Protocol  | Direction | Port-Range   | Purpose                  | Used by
 :---:    | :---:     | :---:        |  :---                    | :---
TCP	      | Inbound	 | 6443          | Kubernetes API server    | kube-apiserver, etcd
TCP	      | Inbound   |	2379-2380	   | etcd server client API	  | client API
TCP	      | Inbound   |	10250	       | Kubelet API	             | Self, Control plane
TCP	      | Inbound   |	10259	       | kube-scheduler	          | Self
TCP	      | Inbound   |	10257	       | kube-controller-manager	 | Self

For the worker nodes:
Protocol  | Direction | Port-Range   | Purpose                  | Used by
 :---:    | :---:     | :---:        |  :---                    | :---
TCP	      | Inbound	  | 10250        | Kubelet API              | Self, Control plane
TCP	      | Inbound   |	10256	       | kube-proxy               | Self, Load balancers
TCP	      | Inbound   |	30000-32767	 | NodePort Services        | All

Firewall Commands:
```bash
sudo ufw status			              # Check firewall status
sudo ufw allow 10250/tcp		      # Open a port 
sudo ufw allow 30000:32767/tcp    # Open a range off ports
```

---

## 3. Disable swap memory
By default, Kubernetes will not start if swap is enabled. Although it’s possible to configure Kubernetes to allow swap, we will disable it entirely in our environment.

> Do this on all nodes!

Disable swap temporarily:
```bash
sudo swapoff -a
```
Disable swap permanently:
Edit the fstab file:
```bash
sudo vi /etc/fstab
```
Comment out the line containing LV_swap, save the file, and reboot the server.
```bash
# <file system>                   <mount point>        <type>     <options>           <dump>  <pass>
/dev/mapper/LV_sys-LV_root        /                    ext4       errors=remount-ro   0       1
/dev/mapper/LV_data-LV_data       /data                xfs        defaults            0       0
/dev/mapper/LV_data-LV_k8s        /var/lib/k8s         xfs        defaults            0       0
/dev/mapper/LV_data-LV_logs       /var/log/containers  xfs        defaults            0       0
# /dev/mapper/LV_sys-LV_swap      none                 swap       sw                  0       0
```

---

## 4. Install Container Runtime
Kubernetes Container Runtimes: https://kubernetes.io/docs/setup/production-environment/container-runtimes/

Containerd is the default container runtime for Kubernetes and has the lowest overhead. That’s why we’ll use it for this setup.

Install containerd:
```bash
sudo apt update
sudo apt install containerd
```
Create config directory (if it doesn’t exist):
```bash
sudo mkdir -p /etc/containerd
```
The `config.toml` file is located in the `/etc/containerd/ directory`. This file must be reviewed and, if necessary, adjusted for Kubernetes compatibility. Use the following command to generate a default configuration as a starting point.

Generate default configuration:
```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
Edit the configuration file:
```bash
sudo vi /etc/containerd/config.toml
```
In the /etc/containerd/config.toml file, the following settings are crucial for Kubernetes compatibility:
```bash
[plugins."io.containerd.grpc.v1.cri"]
   sandbox_image = "registry.k8s.io/pause:3.10.1"

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
   SystemdCgroup = true
```
**Sandbox Image:**
- This refers to the pause container image, which Kubernetes uses to manage the network namespace and shared resources for each pod.
- The pause container acts as the invisible parent of all containers in a pod — it holds the pod's IP address and keeps the namespace alive even if app containers crash or restart.
- The value `registry.k8s.io/pause:3.10.1` points to the official and current version of the pause image. Using the correct version ensures compatibility with your Kubernetes setup and avoids runtime errors.

**SystemdCgroup:**
- This setting tells containerd to use systemd as the cgroup driver instead of the default cgroupfs.
- Kubernetes expects this setting to match the node’s system configuration — especially on Debian 12, which uses systemd by default.
- Mismatched cgroup drivers between containerd and kubelet can lead to pod startup failures or resource management issues.

---

## 5. Enable IP Forwarding
IP forwarding is required for communication within the Kubernetes cluster.

Edit the sysctl configuration:
```bash
sudo vi /etc/sysctl.conf
```
Uncomment the following line:
```bash
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```
Apply the changes:
```bash
sudo sysctl -p
```

> If all steps are completed on every server, the nodes are ready for the next chapter: Installing Kubernetes


