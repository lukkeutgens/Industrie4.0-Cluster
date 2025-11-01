# TLS Trust Chain Setup via cert-manager
We are using cert-manager for managing certificates for our cluster. These are **self-signed**. OAuth2-Proxy needs to trust these certificates so we need to add the root CA to it.

## Check the managed certificates
First let's check what cert-manager has at this moment:
```bash
kubectl get certificates --all-namespaces
```
You'll see something like this
```bash
NAMESPACE              NAME            READY   SECRET         AGE
keycloak               keycloak-cert   True    keycloak-tls   12d
kubernetes-dashboard   kubedash-cert   True    kubedash-tls   47d
longhorn-system        longhorn-cert   True    longhorn-tls   47d
longhorn-system        longhorn-tls    False   longhorn-tls   47d
```
In my case like above, we are missing the root CA certificate. Basically, all certificates are single-self-signed and therefore services will not thrust other services.
Let's fix this by making a root CA certificate and sign all service certificates with it.

## Create root CA certificate
Generate certificate witch CN = Industrie4.0 Root CA for iot.keutgens.be:
```bash
openssl req -x509 -newkey rsa:4096 \
  -keyout root-ca.key \
  -out root-ca.crt \
  -days 3650 -nodes \
  -subj "/CN=Industrie4.0 Root CA for iot.keutgens.be"
```
This will create two CA-certificate files:
- root-ca.crt
- root-ca.key

## Create a Kubernetes secret:
```bash
vi root-ca-secret.yaml
```
With content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: root-ca-secret
  namespace: cert-manager
type: kubernetes.io/tls
data:
  tls.crt: <base64 gecodeerde root-ca.crt>
  tls.key: <base64 gecodeerde root-ca.key>
```
To fill in the <base64> data, we need to generate it from the CA certificate files we created earlier. Use the following commands to get the large base64 decoded strings.
```bash
base64 -w0 root-ca.crt  # voor tls.crt
base64 -w0 root-ca.key  # voor tls.key
```

