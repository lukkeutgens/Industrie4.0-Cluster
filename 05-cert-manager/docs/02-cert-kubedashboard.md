# TLS Certificate for Kubernetes Dashboard
Adding a certificate to the dashboard so we can call it with using "https".

## 1. Create Certificate Resource
Create a file named `kubedash-cert.yaml`:
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: kubedash-cert
  namespace: kubernetes-dashboard
spec:
  secretName: kubedash-tls
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer
  commonName: kubedash.iot.keutgens.be
  dnsNames:
    - kubedash.iot.keutgens.be
  duration: 8760h
  renewBefore: 360h
```
Apply the certificate resource:
```bash
kubectl apply -f kubedash-cert.yaml
```
Verify that the certificate was issued and stored in a Secret:
```bash
kubectl get secret kubedash-tls -n kubernetes-dashboard
```
You should see the kubedash-tls Secret containing:
```yaml
data:
  tls.crt: <base64 encoded certificate>
  tls.key: <base64 encoded private key>
```

---

## 2. Update Ingress for Kubernetes Dashboard
Create or modify `dashboard-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    cert-manager.io/cluster-issuer: "selfsigned-cluster-issuer"
spec:
  tls:
  - hosts:
      - kubedash.iot.keutgens.be
    secretName: kubedash-tls
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
Apply the Ingress configuration:
```bash
kubectl apply -f dashboard-ingress.yaml
```

> ⚠️ Note on Internal Encryption: This setup enables TLS encryption between the browser and the Ingress controller (HTTPS). However, traffic from the Ingress controller to the actual pods (e.g. Kubernetets dashboard backend) still uses unencrypted HTTP.



