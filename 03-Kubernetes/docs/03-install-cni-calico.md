# Install CNI-Calico Plugin
[About Calico](https://docs.tigera.io/calico/latest/about/)

A CNI plugin (Container Network Interface) is a networking component used by Kubernetes to assign IP addresses to pods and enable communication between containers and nodes.  
Without a CNI plugin, pods cannot communicate with each other, and the cluster remains in a `Not Ready` state.

We choose **Calico** because it provides stable networking functionality and supports **Network Policies** — essential for segmenting and securing traffic between services.  
In the context of an Industry 4.0 environment Calico offers:
- Fine-grained control over which services can communicate  
- Observability and logging of network traffic  
- Seamless integration with Kubernetes RBAC and namespaces  

In short: Calico combines lightweight performance with enterprise-grade security — ideal for a learning environment that’s evolving toward production.

[Install Calico on Self-Managed On-Premises Kubernetes](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises)

We’ll install **Calico Open Source** in a Kubernetes cluster running on physical or virtual servers (not cloud-managed).  
Calico is deployed using Kubernetes manifests — YAML files that define resources like Deployments, DaemonSets, and Custom Resource Definitions (CRDs).  
There is no separate “image” to download outside of Kubernetes.

---

## 1. Install Calico via Tigera Operator
### Download and rename the manifests:
```bash
# Operator CRDs
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/operator-crds.yaml
mv operator-crds.yaml 01-operator-crds.yaml

# Tigera Operator
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/tigera-operator.yaml
mv tigera-operator.yaml 02-tigera-operator.yaml

# Calico Custom Resources
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.30.3/manifests/custom-resources.yaml
mv custom-resources.yaml 03-custom-resources.yaml
```

---

## 2. Edit the Calico Custom Resources
Open the file:
```bash
vi 03-custom-resources.yaml
```
Update the following section to match your network setup:
```yaml
spec:
  calicoNetwork:
    bgp: Disabled
    nodeAddressAutodetectionV4:
      interface: enp0s3
    ipPools:
    - cidr: 192.168.0.0/16
      encapsulation: VXLAN
      natOutgoing: Enabled
      blockSize: 26
      nodeSelector: all()
```
> We use 192.168.0.0/16 because it matches the pod CIDR set during kubeadm init and is the default for Calico.

---

## 3. Apply the Manifests
```bash
kubectl apply -f 01-operator-crds.yaml
kubectl apply -f 02-tigera-operator.yaml
kubectl apply -f 03-custom-resources.yaml
```

---

## 4. Optional: Debug and Reapply Installation
> ⚠️ During our first installation, we had to debug and reapply the configuration. This step is usually not required, but can help if Calico pods fail to start.

Export the current installation:
```bash
kubectl get installation.operator.tigera.io default -o yaml > operator-installation.yaml
```
Edit the file:
```bash
vi operator-installation.yaml
```
Update the spec section:
```yaml
spec:
  calicoNetwork:
    bgp: Disabled
    encapsulation: VXLAN
    nodeAddressAutodetectionV4:
      interface: enp0s3
    ipPools:
    - name: default-ipv4-ippool
      cidr: 192.168.0.0/16
      encapsulation: VXLAN
      natOutgoing: Enabled
      blockSize: 26
      nodeSelector: all()
```
Apply the updated configuration:
```bash
kubectl apply -f operator-installation.yaml
```

---

## 5. Network Requirements and Firewall Settings
> ⚠️ This step must be applied to every node (control-plane and workers)

Calico requires specific ports for control-plane communication and dataplane traffic. 
We use VXLAN mode because we’re working with virtual machines — BGP is not needed.

Open UFW ports (on all nodes):
```bash
# Typha (Felix sync)
sudo ufw allow 5473/tcp

# Kubelet API (node status, metrics)
sudo ufw allow 10250/tcp

# NodePort services (optional, for external access)
sudo ufw allow 30000:32767/tcp

# Remove BGP port
sudo ufw delete allow 179/tcp
```

Adjust UFW Forward Policy
VXLAN tunnels and ClusterIP traffic require UFW to allow forwarded packets.

> ⚠️ At first I didn't do this step and took me hours to find out why Calico would always fail.

Edit the file:
```bash
sudo vi /etc/default/ufw
```
Change the following line:
```bash
DEFAULT_FORWARD_POLICY="ACCEPT"
```
Reload UFW:
```bash
sudo ufw reload
```

---

## 6. Set Netfilter Sysctls
> ⚠️ This step must be applied to every node (control-plane and workers)

These settings ensure that iptables rules are correctly applied to bridged interfaces (like vxlan.calico). 

```bash
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.bridge.bridge-nf-call-iptables=1
sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1
```

---

## 7. Restart Calico Pods and Fix Kube-Proxy
Sometimes pods need to be restarted manually after installation.
Restart Calico pods:
```bash
kubectl -n calico-system delete pod -l k8s-app=calico-node
kubectl -n calico-system delete pod -l k8s-app=calico-kube-controllers
```
Restart kube-proxy:
```bash
kubectl -n kube-system delete pod -l k8s-app=kube-proxy
```
Once Calico is installed and pods are restarted, your cluster should transition to Ready status and pod networking will be functional.

---

## 8. Control Plane Is Ready
With Calico successfully installed as the CNI plugin, your control plane node should now be fully operational.

Verify the node status:
```bash
kubectl get nodes
```
Expected output:
```bash
NAME     STATUS   ROLES           AGE     VERSION
csrv1    Ready    control-plane   XXm     v1.34.x
```

The node csrv1 should now show Ready status. 
This confirms that networking is functional and the cluster is no longer blocked by missing CNI configuration.

