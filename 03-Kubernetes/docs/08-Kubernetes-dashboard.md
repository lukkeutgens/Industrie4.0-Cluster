# Kubernetes Dashboard
[Official Documentation](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

The Kubernetes Dashboard is a web-based UI for managing cluster resources, viewing workloads, and monitoring performance.

---

## 1. Install the Dashboard
Install the dashboard using Helm:
```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --create-namespace --namespace kubernetes-dashboard
```

---

## 2. Expose the Dashboard via Ingress
We’ll expose the dashboard using the NGINX Ingress Controller and a custom domain.

### 2.1 Delete the default service (if present)
```bash
kubectl delete svc kubernetes-dashboard-kong-proxy -n kubernetes-dashboard
```

### 2.2 Create a new ClusterIP service.
Create a file named `dashboard-web-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  selector:
    app.kubernetes.io/name: kubernetes-dashboard-web
  ports:
  - port: 443
    targetPort: 8000
    protocol: TCP
    name: https
  type: ClusterIP
```
Apply the service:
```bash
kubectl apply -f dashboard-web-service.yaml
```

### 2.3 Create the Ingress resource
Create a file named `dashboard-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
spec:
  rules:
  - host: kubedash.iot.keutgens.be
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```
Apply the Ingress:
```bash
kubectl apply -f dashboard-ingress.yaml
```
You can now access the dashboard at: https://kubedash.iot.keutgens.be

---


## 3. Create an Admin Service Account
To access the dashboard, create a temporary admin user.
Create a file named admin-user.yaml:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: iotadmin
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: iotadmin
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: iotadmin
  namespace: kubernetes-dashboard
```
Apply the file:
```bash
kubectl apply -f admin-user.yaml
```
Generate a login token:
```bash
kubectl -n kubernetes-dashboard create token iotadmin
```
Use this token to log in to the dashboard.
> ⚠️ The dashboard is stateless and does not store user sessions. You’ll need to re-enter the token each time.
> In a future phase, we’ll switch to OpenID Connect (OIDC) authentication using a provider like Keycloak.

---

## 4. Install Metrics Server
[Metrics Server](https://kubernetes.io/docs/concepts/cluster-administration/system-metrics/)
The Metrics Server enables resource usage metrics (CPU, memory) in the dashboard and via kubectl top.

Download and edit the manifest:
```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O metrics-server.yaml
```
Edit the deployment section to include the following args:
```yaml
spec:
  containers:
  - args:
    - --cert-dir=/tmp
    - --secure-port=10250
    - --kubelet-insecure-tls
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --kubelet-use-node-status-port
    - --metric-resolution=15s
    image: registry.k8s.io/metrics-server/metrics-server:v0.8.0
```
Apply the manifest:
```bash
kubectl apply -f metrics-server.yaml
```
Verify metrics:
```bash
kubectl top nodes
kubectl top pods --all-namespaces
```
Metrics may take a minute to appear in the dashboard. Refresh if needed. You can also type namespace names directly in the selector if they don’t appear in the dropdown.

