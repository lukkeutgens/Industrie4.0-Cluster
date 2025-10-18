# Cert-Manager Installation
Follow the official Helm instructions: https://cert-manager.io/docs/installation/helm

## 1. Install via Helm
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```
You can then install cert-manager using the Helm chart (see official docs for version pinning and CRDs).

---

## 2. Create a Self-Signed ClusterIssuer
Create a file named `clusterissuer-selfsigned.yaml`:
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-cluster-issuer
spec:
  selfSigned: {}
```
Apply the file:
```bash
kubectl apply -f clusterissuer-selfsigned.yaml
```
This sets up a basic self-signed certificate authority for internal use — useful for testing or internal services without public CA requirements.

> ⚠️ Note: This guide reflects the exact steps I followed during my Kubernetes project. The descriptions may differ slightly from the files/ section, which likely contains the most up-to-date version.

