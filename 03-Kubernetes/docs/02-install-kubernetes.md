# Installing Kubernetes
- **Kubeadm** : The command-line tool used to bootstrap and initialize your Kubernetes cluster.
- **Kubelet** : The agent that runs on every machine in the cluster and is responsible for starting pods and containers.
 -**Kubectl** : The command-line tool used to interact with and manage your Kubernetes cluster.

---

## 1. Install Required Packages
> ⚠️ The following steps must be executed on all servers. That said, installing kubectl on worker nodes is optional — but in our case, we’ll include it on all nodes for convenience.

First, install some additional packages required for accessing the Kubernetes repository:
- `apt-transport-https`: Enables downloading packages over HTTPS (note: sometimes a dummy package in Debian 12)
- `ca-certificates`: Ensures HTTPS certificate verification works correctly
- `curl`: Used to download files from the web
- `gpg`: Used to verify cryptographic signatures

### 1.1 Update package index:
```bash
sudo apt-get update
```
### 1.2 Install required packages:
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
Check first if de folder `/etc/apt/keyrings/` exists, if not make it.

### 1.3 Download the Kubernetes GPG key:
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
### 1.4 Add the Kubernetes repository:
> ⚠️Remark: This will overwrite any existing configuration in `/etc/apt/sources.list.d/kubernetes.list`
```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
The above will only use version 1.34

Verify the repository configuration:
```bash
cat /etc/apt/sources.list.d/kubernetes.list
```
Expected output:
```bash
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ 
```

---

## 2. Install Kubelet, Kubeadm and Kubectl
Now let's install kubernetes:
- **kubelet** : Runs on every node and manages the containers
- **kubeadm** : Tool to setup the cluster
- **kubectl** : CLI to setup the cluster

> ⚠️ The following steps must be executed on all servers.

### 2.1 Install packages:
```bash
sudo apt-get update
sudo apt-get install kubelet kubeadm kubectl
```

### 2.2 Lock versions
Prevents these packages from being automatically upgraded during an apt upgrade. Important for maintaining version compatibility within your cluster.
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```
Check with:
```
Check met: apt-mark showhold
```
Expected output should be: kubeadm, kubectl and kubelet

### 2.3 Enable the Kubelet Service
Before running kubeadm, you need to enable the kubelet service:
```bash
sudo systemctl enable --now kubelet
```
This starts the kubelet and ensures it runs automatically on boot. At this stage, the kubelet will enter a crash loop, restarting every few seconds. This is expected behavior — the kubelet is waiting for instructions from kubeadm to know how to configure and manage the node.

> This step ensures that the kubelet is active and ready to receive its configuration once kubeadm init or kubeadm join is executed.

---

## 3. Initialise with Kubeadm
> ⚠️ Next steps are only needed on the control plane server

### 3.1 Reset active cluster (optional):
```bash
sudo kubeadm reset
```

### 3.2 Initialize the Kubernetes Control Plane
Before initializing the cluster, it’s important to use the correct network settings. Kubernetes creates its own internal network to allow communication between pods, so the pod IP range must be chosen carefully. To ensure external connectivity, the correct network interface and default route must also be selected.

You can inspect your network routes with the following command:
```bash
ip route show
```
You should see something like:
```bash
default via 192.168.50.1 dev enp0s3 onlink
```
This indicates the default route and the active network interface.

In addition, you must choose a free IP range for the pod network — one that does not overlap with your existing LAN or VPN ranges.

Initialize the cluster:
```bash
sudo kubeadm init --apiserver-advertise-address=192.168.50.160 --pod-network-cidr=192.168.0.0/16
```
This command initializes kubeadm using the server’s advertised IP address and sets the pod network CIDR to a non-conflicting range. We specifically choose `192.168.0.0/16` because it’s the default range used by Calico, which we’ll install later as our CNI plugin.

### 3.3 Configure kubectl for the Current User
To use kubectl as a non-root user, you need to configure access to the cluster.
Create the .kube directory:
```bash
mkdir /home/<username>/.kube
```
> This is a hidden directory inside the user's home folder.

Copy the admin configuration file:
```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
> ⚠️ Note: In the files section, the `config` file was renamed to `kubeconfig-csrv1` for clarity and node identification.

Set the correct ownership:
```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 4. Test the Setup
Verify that kubectl is working by listing all pods across all namespaces:
```bash
kubectl get pods --all-namespaces
```
You should see output from the control plane node. At this stage, the node will not yet be marked as Ready — this is expected. We still need to install a CNI plugin (such as Calico) to enable pod networking.



