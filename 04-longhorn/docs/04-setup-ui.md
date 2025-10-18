# Exposing the Longhorn UI via Ingress
Longhorn includes a built-in web interface for managing volumes, nodes, backups, and settings. We will expose this UI using an Ingress resource so it becomes accessible via a domain name.

While Longhorn can be managed via kubectl and CRDs, the web interface is significantly easier to use — especially for visualizing volume health, replication status, and troubleshooting.

> ⚠️ Note: This guide reflects the exact steps I followed during my Kubernetes project. The descriptions may differ from the files section, which likely contains the most up-to-date version.

---

## 1. Create Ingress Resource
Create a file named `longhorn-ingress.yaml` with the following content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ui
  namespace: longhorn-system
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - host: longhorn.iot.keutgens.be
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
```
Apply the file:
```bash
kubectl apply -f longhorn-ingress.yaml
```

---

## 2. Add DNS Entry to CoreDNS
Update your custom CoreDNS deployment to include the new domain. Edit `coredns-custom-depl.yaml` and modify the `Corefile` section:
```yaml
data:
  Corefile: |
    .:53 {
      hosts {
        192.168.50.210 kubedash.iot.keutgens.be
        192.168.50.210 grafana.iot.keutgens.be
        192.168.50.210 longhorn.iot.keutgens.be    # ← Add this line
        fallthrough
      }
      forward . 8.8.8.8
      log
    }
```
Apply the updated CoreDNS config:
```bash
kubectl apply -f coredns-custom-depl.yaml
```
Restart the CoreDNS pods to apply changes:
```bash
kubectl rollout restart deployment coredns-custom -n kube-system
```

---
Once complete, you can access the Longhorn UI at: http://longhorn.iot.keutgens.be
