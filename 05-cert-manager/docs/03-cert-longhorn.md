# TLS Certificate for Longhorn UI
Create a certificate to add to the Longhorn storage UI so we can access it using "https".

---

## 1. Create Certificate Resource
Create a file named `longhorn-cert.yaml`:
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: longhorn-cert
  namespace: longhorn-system
spec:
  secretName: longhorn-tls
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer
  commonName: longhorn.iot.keutgens.be
  dnsNames:
    - longhorn.iot.keutgens.be
  duration: 8760h
  renewBefore: 360h
```
Apply the certificate:
```bash
kubectl apply -f longhorn-cert.yaml
```
Verify the Secret:
```bash
kubectl get secret longhorn-tls -n longhorn-system
```
You should see the longhorn-tls Secret containing:
```yaml
data:
  tls.crt: <base64 encoded certificate>
  tls.key: <base64 encoded private key>
```

---

## 2. Configure Ingress for Longhorn UI
Create or modify `longhorn-ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ui
  namespace: longhorn-system
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    cert-manager.io/cluster-issuer: "selfsigned-cluster-issuer"
spec:
  tls:
  - hosts:
    - longhorn.iot.keutgens.be
    secretName: longhorn-tls
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
Apply the Ingress:
```bash
kubectl apply -f longhorn-ingress.yaml
```
> ⚠️ Note on Internal Encryption: This setup enables TLS encryption between the browser and the Ingress controller (HTTPS). However, traffic from the Ingress controller to the actual pods (e.g. Longhorn UI backend) still uses unencrypted HTTP.





