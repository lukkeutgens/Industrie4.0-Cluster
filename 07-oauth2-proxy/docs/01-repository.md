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
  clientID: "placeholder for Keycloak ID"
  clientSecret: "placeholder for Keycloak secret"
  cookieSecret: "cookieSecret created above"
```

Apply it with:
```bash
kubectl apply -f oauth2-secret.yaml
```
This secret will be referenced in the Helm chart configuration to inject credentials securely into the OAuth2 Proxy container.

---

## 3. Create values.yaml for OAuth2 Proxy
The values.yaml file defines how OAuth2 Proxy will behave inside your cluster. It includes configuration for the identity provider (Keycloak), upstream service (Kubernetes Dashboard), cookie settings, and Ingress routing. We will use a single configuration (values.yaml) for each service we want to link up like `Kubernetes-Dashboard` and install it in that namespace.

> 🔒 At this stage, Keycloak is not yet configured. We use placeholder values for clientID and clientSecret, which will be updated later.

Create the file:
```bash
vi oauth2-dashboard-values.yaml
```
Add the content:
```yaml
config:
  existingSecret: oauth2-proxy-secret
  configFile: |
    provider = "oidc"
    oidc_issuer_url = "https://keycloak.iot.keutgens.be/realms/iot"
    client_id = "use-from-secret"
    client_secret = "use-from-secret"
    cookie_secret = "use-from-secret"
    email_domains = [ "*" ]
    whitelist_domains = [ "iot.keutgens.be" ]
    upstreams = [ "http://kubernetes-dashboard.kube-system.svc.cluster.local:80" ]
    scope = "openid profile email"
    cookie_secure = false
    cookie_samesite = "lax"
    cookie_domains = [ "iot.keutgens.be" ]
    pass_access_token = true
    set_authorization_header = true
    set_xauthrequest = true

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: "nginx"
  hosts:
    - oauth2.iot.keutgens.be
  tls:
    - hosts:
        - oauth2.iot.keutgens.be
      secretName: oauth2-proxy-cert
```
Key Sections Explained:
- config.existingSecret: References the Kubernetes secret created in Step 2
- oidc_issuer_url: Points to your Keycloak realm (placeholder for now)
- upstreams: Targets the internal Kubernetes Dashboard service
- cookie_domains and whitelist_domains: Restrict session scope to your domain
- ingress: Exposes OAuth2 Proxy via NGINX Ingress on oauth2.iot.keutgens.be
- use-secret-from-k8s: Will tell the chart will inject it correctly

Later, when Keycloak is configured, you will replace the placeholders with real values and validate the authentication flow.



