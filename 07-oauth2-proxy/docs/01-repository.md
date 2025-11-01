# OAuth2-Proxy Helm Repository
To install OAuth2 Proxy in your Kubernetes cluster, we use the official Helm chart provided by the [OAuth2 Proxy manifests repository](https://github.com/oauth2-proxy/manifests) project. This Helm chart simplifies deployment and allows for easy configuration via values.yaml.

---

## 1. Add the Helm Repository
```bash
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests
helm repo update
```
This makes the latest version of the oauth2-proxy chart available for installation.

---

## 2. Create Secrets for OAuth2 Proxy
OAuth2 Proxy requires sensitive credentials to authenticate users via an identity provider (e.g., Keycloak). These credentials must be securely stored in Kubernetes as a Secret, which the Helm chart can reference during deployment.

### Required Secrets
Before installing OAuth2 Proxy, prepare the following values:
- **clientID**: The OAuth2 client ID (to be generated later when Keycloak is configured)
- **clientSecret**: The corresponding client secret
- **cookieSecret**: A random 32-byte string used to encrypt session cookies

The cookieSecret must be a base64-encoded 32-byte string. You can generate one using:
```bash
 openssl rand -base64 32 | head -c 32 | base64
```

### Create the Kubernetes Secret
Create the secret file:
```bash
vi oauth2-secret.yaml
```
Add the content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: oauth2-proxy-secret
  namespace: default
type: Opaque
stringData:
  clientID: "<your-client-id>"
  clientSecret: "<your-client-secret>"
  cookieSecret: "<your-cookie-secret>"
```
Apply it with:
```bash
kubectl apply -f oauth2-secret.yaml
```
