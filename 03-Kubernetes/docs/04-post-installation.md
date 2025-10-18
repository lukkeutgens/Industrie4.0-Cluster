# Post Installation Steps
After these steps our standard Kubernetes cluster should be up and running on all our 4 nodes/servers, ready to add all the services we want.

---

## 1. Add Worker Nodes

Now that the control plane is ready, we can add the worker nodes to the cluster.

On the **control plane node**, run the following command to generate a join command:
```bash
kubeadm token create --print-join-command
```
Example output:
```bash
kubeadm join 192.168.50.160:6443 --token 2lp1bn.573blabla3bqpk094 --discovery-token-ca-cert-hash sha256:b1140d8611e0balalral10b81106c5db827127d646f0ab
```
> ⚠️ Use the actual output from your system — not the example above.

Run the generated command on each worker node, prefixed with sudo:
```bash
sudo kubeadm join ...
```
To verify that the nodes have joined successfully, run:
```bash
kubectl get nodes
```
You should now see multiple nodes listed, including the newly joined workers.

---

## 2. Remove Taint from Control Plane Node
In Kubernetes, a taint is a label applied to a node that prevents regular pods from being scheduled there — unless those pods explicitly tolerate the taint.
- Taint → applied to the node: "Only pods that tolerate me may run here."
- Toleration → applied to the pod: "I can run on nodes with this taint."

A taint consists of:
- Key: e.g. node-role.kubernetes.io/control-plane
- Value: optional (e.g. true)
- Effect:
    - NoSchedule: do not schedule pods here
    - PreferNoSchedule: avoid scheduling unless necessary
    - NoExecute: evict pods that don’t tolerate the taint

We’ll remove the default taint from the control plane node so it can also run regular workloads — useful in resource-constrained environments.

Check for taints:
```bash
kubectl describe node csrv1 | grep -i taint
Expected output:
```
```bash
Taints: node-role.kubernetes.io/control-plane:NoSchedule
```
Remove the taint:
```bash
kubectl taint nodes csrv1 node-role.kubernetes.io/control-plane:NoSchedule-
```
The trailing "-" removes the taint.

---

## 3. Install Helm Package Manager
Helm is the most widely used package manager for Kubernetes. Before installing, make sure you're in a temporary working directory.

Download and install Helm:
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
Verify installation:
```bash
helm version
```

With these steps completed, our cluster is now ready for deploying workloads and managing applications with Helm.

