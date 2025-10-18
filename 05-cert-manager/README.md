# Cert-Manager
Website: https://cert-manager.io

TLS (Transport Layer Security) encrypts network traffic between clients (e.g. browsers) and services, ensuring:
- Confidentiality: Sensitive data (e.g. tokens, credentials) cannot be intercepted or read.
- Integrity: Data cannot be altered during transit without detection.
- Authentication: Clients can verify they’re communicating with a trusted domain (e.g. *.iot.keutgens.be).

Without TLS, services are exposed over plain HTTP, which is insecure, vulnerable to man-in-the-middle attacks, and often blocked by modern browsers and security policies.

**cert-manager** automates the creation, renewal, and management of TLS certificates for workloads running in your Kubernetes or OpenShift cluster.
It supports multiple issuers (e.g. Let's Encrypt, Vault, self-signed) and integrates with Ingress, webhooks, and internal services — making secure communication easier to maintain.

When you create a Certificate resource, cert-manager automatically generates:
- A private key
- A certificate signing request (CSR)
- A full X.509 certificate

These are stored in a Kubernetes Secret, using the name specified in spec.secretName.
You can reference this Secret in your Ingress, Deployment, or other workloads to enable HTTPS or mTLS.

Why Secrets instead of Persistent Storage?
- Secrets are native Kubernetes resources and automatically synced to nodes where needed.
- They are encrypted in etcd (if encryption-at-rest is configured).
- cert-manager uses these Secrets as the source for TLS in Ingress or for mounting into pods.
- The Secret is automatically created by cert-manager once the certificate is successfully issued.

---

## Example:
If you define:
```yaml
spec:
  secretName: dashboard-tls
```
Then you can retrieve the certificate with:
```bash
kubectl get secret dashboard-tls -n kubernetes-dashboard -o yaml
```
Inside the Secret you'll find:
```yaml
data:
  tls.crt: <base64 encoded certificate>
  tls.key: <base64 encoded private key>
```

---

## Folder Contents
> ⚠️ Note: This guide reflects the exact steps I followed during my Kubernetes project. The descriptions may differ slightly from the files/ section, which likely contains the most up-to-date version.

```plaintext
05-cert-manager/
├── docs
│   ├── 01-cert-install.md               # Install cert-manager via Helm
│   ├── 02-cert-kubedashboard.md         # Install TLS for Kubernetes Dashboard
│   └── 03-cert-longhorn.md              # Install TLS for Longhorn UI
├── files
│   ├── clusterissuer-selfsigned.yaml    # Self-signed ClusterIssuer
│   ├── kubedash-cert.yaml               # Certificate for Dashboard
│   └── longhorn-cert.yaml               # Certificate for Longhorn
└── README.md                            # This file
```




