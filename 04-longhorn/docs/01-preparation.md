# Preparing Debian Nodes for Longhorn
Before deploying Longhorn, each node must be prepared with the required services, kernel modules, and system utilities.
This section outlines the manual steps, although Longhorn also provides a CLI tool to automate the preflight setup:
```bash
longhornctl --kube-config ~/.kube/config \
  --image longhornio/longhorn-cli:v1.9.1 \
  install preflight
```
In this guide, we perform each step manually. The automated method via `longhornctl` was not tested in this setup.

---

## 1. Create Namespace
Longhorn components are deployed into the `longhorn-system` namespace. To avoid installation errors, we create it manually:
```bash
kubectl create namespace longhorn-system
```
> ⚠️ Tip: Pre-creating the namespace avoids race conditions and ensures all resources deploy correctly.

---

## 2. Install open-iscsi
> ➡️ This will install on all nodes through Kubernetes, so this step is only for the control-plane.

Longhorn requires `open-iscsi` to manage block storage. Apply the official YAML to install it across all nodes:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.9.1/deploy/prerequisite/longhorn-iscsi-installation.yaml
```
Verify that the installation pods are running:
```bash
kubectl -n longhorn-system get pod | grep longhorn-iscsi-installation
```
Expected output:
```bash
longhorn-iscsi-installation-49hd7   1/1     Running   0          21m
longhorn-iscsi-installation-pzb7r   1/1     Running   0          39m
```
These pods install and maintain open-iscsi on each node.

> ⚠️ Issue: On first install, the control plane node `csrv1` had a NoSchedule taint, preventing the `iscsi` pod from running. See Kubernetes taint removal for resolution.

---

## 3. Install NFSv4 Client
> ➡️ Do this step on all nodes!

Longhorn uses `NFSv4` for backup targets and for RWX (ReadWriteMany) volumes, which are essential for multi-node access.
Longhorn backups and RWX volumes require `NFSv4.1` or higher. 
First, verify kernel support:
```bash
cat /boot/config-$(uname -r) | grep CONFIG_NFS_V4_1
cat /boot/config-$(uname -r) | grep CONFIG_NFS_V4_2
```
Expected output:
```bash
CONFIG_NFS_V4_1=y
CONFIG_NFS_V4_1_IMPLEMENTATION_ID_DOMAIN="kernel.org"
# CONFIG_NFS_V4_1_MIGRATION is not set
```
Install the required utilities:
```bash
sudo apt update
sudo apt-get install nfs-common
```
Apply the Longhorn NFS installer:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.9.1/deploy/prerequisite/longhorn-nfs-installation.yaml
```

---

## 4. Install Cryptsetup and LUKS2 Support
> ➡️ Do this step on all nodes!

Longhorn supports volume encryption using `LUKS2`. To enable this, install `cryptsetup` on all nodes:
```bash
sudo apt update
sudo apt-get install cryptsetup
sudo cryptsetup --version
```
Ensure the `dm_crypt` kernel module is loaded and persistent:
```bash
sudo modprobe dm_crypt
echo dm_crypt | sudo tee /etc/modules-load.d/dm_crypt.conf
lsmod | grep dm_crypt
```
> ⚠️ If you don’t plan to use encrypted volumes, this step is optional.

---

## 5. Install Device Mapper Userspace Tool
> ➡️ Do this step on all nodes!

The device mapper is a kernel framework used by `dm-crypt` and Longhorn’s volume abstraction. It’s usually preinstalled, but verify and install if needed:
```bash
ls -l /dev/mapper        # Optional check
sudo apt-get install dmsetup
```
> ⚠️ In our setup, `dmsetup` was already present.

---

## 6. Fix DNS Resolution
> ➡️ Do this step on all nodes!

After installing Longhorn, pods failed to connect to the cluster due to DNS issues. 
This issue may manifest as pods stuck in ContainerCreating or CrashLoopBackOff due to failed DNS resolution.

To resolve this, edit `/etc/resolv.conf` on each node:
```bash
sudo vi /etc/resolv.conf
```
Recommended contents:
```bash
nameserver 10.96.0.10
nameserver 1.1.1.1
nameserver 8.8.8.8
search cluster.local svc.cluster.local
```
10.96.0.10 is the default CoreDNS service IP in many Kubernetes setups. Adjust if your cluster uses a different range.

---

If all steps are done on all the nodes, we are ready for pre-checking them.

