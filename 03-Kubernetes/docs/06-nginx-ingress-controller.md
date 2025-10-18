# Install NGINX Ingress Controller
[Official Deployment Guide](https://kubernetes.github.io/ingress-nginx/deploy/)  
[GitHub Repository](https://github.com/kubernetes/ingress-nginx)

An **Ingress Controller** is a Kubernetes component that:

- Receives external traffic (typically HTTP/HTTPS)  
- Routes requests to the correct service based on Ingress rules  
- Optionally handles TLS (HTTPS) and authentication  
- Allows multiple apps to share a single IP or domain via path-based or host-based routing

## Why NGINX?
NGINX is by far the most widely used Ingress Controller. It’s:
- Stable and well-documented  
- Supports TLS, basic auth, rate limiting, rewrite rules, and more  
- Works seamlessly on bare-metal, cloud platforms, and virtual environments — including our VirtualBox cluster

---

## 1. Add the Helm Repository
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

---

## 2. Create a Custom Values File
Create a file named `ingress-nginx-values.yaml` in your designated configuration directory:
```yaml
controller:
  replicaCount: 2
  ingressClassResource:
    name: nginx
    enabled: true
    default: true
  service:
    type: LoadBalancer
    loadBalancerIP: 192.168.50.210
  config:
    use-forwarded-headers: "true"
    proxy-body-size: "20m"
    enable-real-ip: "true"
  metrics:
    enabled: true
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 512Mi
  defaultBackend:
    enabled: true
```
This configuration sets up a LoadBalancer service using MetalLB, enables metrics, and configures resource limits and headers for real IP forwarding.


---

## 3. Install the Ingress Controller
```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --values ingress-nginx-values.yaml
```
Once installed, you can start creating Ingress resources to expose your services via HTTP/HTTPS. 
NGINX will route traffic based on hostnames or paths, and you can enable TLS termination, authentication, and more.







