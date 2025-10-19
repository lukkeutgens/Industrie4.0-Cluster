# Keycloak Certificate
Create a certificate with cert-manager for Keycloak

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
