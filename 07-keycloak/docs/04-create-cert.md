# TLS Certificate for Keycloak via cert-manager
To enable HTTPS for Keycloak, create a TLS certificate using cert-manager and a `ClusterIssuer`, which we already have setup.

Create file:
```bash
vi keycloak-cert.yaml
```
Edit content:
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: keycloak-cert
  namespace: keycloak
spec:
  secretName: keycloak-tls
  issuerRef:
    name: selfsigned-cluster-issuer
    kind: ClusterIssuer
  commonName: keycloak.iot.keutgens.be
  dnsNames:
    - keycloak.iot.keutgens.be
  duration: 8760h
  renewBefore: 360h
```
Apply the file:
```bash
kubectl apply -f keycloak-cert.yaml
```
The certificate will be stored as a Kubernetes TLS secret named keycloak-tls in the keycloak namespace.

Make sure the ClusterIssuer named selfsigned-cluster-issuer already exists. You can verify this with:
```bash
kubectl get clusterissuer
```
