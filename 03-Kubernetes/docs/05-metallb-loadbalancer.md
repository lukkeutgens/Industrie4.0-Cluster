# Install MetalLB Load Balancer
[MetalLB Official Website](https://metallb.io/)

MetalLB is a load balancer implementation for **bare-metal Kubernetes clusters**, using standard routing protocols.

By default, Kubernetes does **not** provide a built-in implementation of `LoadBalancer` services for bare-metal environments.  
The built-in `LoadBalancer` type only works on cloud platforms like GCP, AWS, or Azure — where it delegates to the provider’s native load balancer.  
On bare-metal, `LoadBalancer` services remain stuck in the `Pending` state indefinitely.

This leaves bare-metal operators with two limited options:
- `NodePort`: exposes services on a random high port on each node
- `externalIPs`: requires manual IP assignment and routing

Both are suboptimal for production use.

**MetalLB** solves this by providing a native load balancer for bare-metal clusters.  
It integrates with your existing network infrastructure and makes `LoadBalancer` services work as expected — even without a cloud provider.

---

## 1. Install MetalLB with Helm
```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update
helm install metallb metallb/metallb -n metallb-system --create-namespace
```

---

## 2. Create IPAddressPool Configuration
Create a file named `metallb-ip-pools.yaml` with the following content:
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: lb-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.50.210-192.168.50.220
```
This defines a pool of IP addresses that MetalLB can assign to LoadBalancer services. Make sure the range is:
- Within your LAN subnet
- Not already in use by DHCP or static devices

---

## 3. Create L2Advertisement Configuration
Create a file named `metallb-advertisement.yaml` with the following content:
```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: lb-adv
  namespace: metallb-system
spec:
  ipAddressPools:
    - lb-pool
```
This tells MetalLB to advertise the IPs from the pool using Layer 2 (ARP) — ideal for simple home labs and flat networks.


---

## 4. Apply the Configuration and Verify
Apply both configuration files:
```bash
kubectl apply -f metallb-ip-pools.yaml
kubectl apply -f metallb-advertisement.yaml
```
Check that the MetalLB pods are running:
```bash
kubectl get pods -n metallb-system
```
MetalLB is now installed and ready. 
You can now create Kubernetes services of type LoadBalancer, and MetalLB will assign them an IP from your configured pool.

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```






