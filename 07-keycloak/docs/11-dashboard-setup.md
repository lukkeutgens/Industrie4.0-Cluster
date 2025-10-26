# Kubernetes-Dashboard Login Setup
> Please read also `10-kubernetes-RBAC` to better understand the setup we're implementing.

This document describes how to use Keycloak to enable secure login via OpenID Connect (OIDC) for the Kubernetes Dashboard.

We will define Roles, Groups, and Users in Keycloak to control access levels within the dashboard.

---

## 1. Create Keycloak Realm Roles
### Viewer role:
- **Role name** : k8s-viewer
- **Description** : Read-only access to Kubernetes Dashboard
- **Attributes** :

| key          | Value              |
| :---         | :---               |
| apiGroups    | ["", "apps", "batch", "networking.k8s.io"]  |
| resources    | ["pods", "services", "events", "deployments", "replicasets", "statefulsets", "jobs", "cronjobs", "ingresses"]  |
| verbs        | ["get", "list", "watch"]  |
| description  | Read-only access to Kubernetes Dashboard |

The viewer role receives access to specific Kubernetes resources because the Dashboard displays information derived from those resources. Without read permissions (get, list, watch) on these resources, the Dashboard would not be able to render their content, resulting in missing sections or access denied errors.

### Settings role:
- **Role name** : k8s-settings
- **Description** : Limited access to dashboard settings
- **Attributes** :

| key          | Value              |
| :---         | :---               |
| apiGroups    | ["", "apps", "networking.k8s.io"]  |
| resources    | ["configmaps", "secrets", "ingresses"]  |
| verbs        | ["get", "list", "create", "update", "patch", "delete"]  |
| description  | Limited access to dashboard settings | 

Note that the k8s-settings role targets a different set of resources than the k8s-viewer role. While both roles include read-only verbs like get and list, the k8s-settings role also allows modification actions such as create, update, and delete — but only on configuration-related resources.

In Keycloak, we assign both roles (k8s-viewer and k8s-settings) to the k8s-settings group. This means users in that group can view all workloads and cluster activity via the k8s-viewer role, and additionally manage specific settings like configmaps, secrets, and ingresses via the k8s-settings role.

### Admin role:
- **Role name** : k8s-admin
- **Description** : Full administrative access to all Kubernetes resources and actions
- **Attributes** :

| key          | Value              |
| :---         | :---               |
| apiGroups    | ["*"]  |
| resources    | ["*"]  |
| verbs        | ["*"]  |
| description  | Limited access to dashboard settings | 

The k8s-admin role is granted full access to all Kubernetes resources and actions. This is achieved by using the wildcard * for apiGroups, resources, and verbs. In contrast, we avoid using wildcards in the k8s-viewer role to prevent exposing sensitive resources such as secrets and other critical configuration data.

---

## 2. Create Keycloak User Groups
We will now create the user groups that control access to the Kubernetes Dashboard. The parent group is k8s-dashboard, with three child groups as shown below:

| User Group      | Child Group | Roles       |
| :---            | :---        | :---        |
| k8s-dashboard   | viewers     | `k8s-viewer`  | 
| k8s-dashboard   | settings    | `k8s-viewer`, `k8s-settings` |
| k8s-dashboard   | admins      | `k8s-admin` |

Each child group receives the appropriate Keycloak roles. These roles determine what the user can see or manage in the dashboard.

To match Keycloak groups in Kubernetes RBAC, use the full group path as seen in the JWT token. For example:
```yaml
subjects:
  - kind: Group
    name: /k8s-dashboard/viewer
    apiGroup: rbac.authorization.k8s.io
```

--- 

## 3. Create Keycloak Users
We will now create three users: one for admin access, one for settings management, and one with viewer-only access.
> ⚠️ When using **child groups**, you must assign users directly from the child group page in Keycloak. It is not possible to assign a user to a child group from the user edit screen — there you can only select top-level groups.

| User      | User Group                |   Roles (auto from group)  |
| :---      | :---                      | :---        |
| iotadmin  | `k8s-dashboard/admins`    | `k8s-admin`  |
| luk       | `k8s-dashboard/settings`  | `k8s-viewer`, `k8s-settings` |
| leen      | `k8s-dashboard/viewers`   | `k8s-viewer`  | 

Settings for a user:
- **Username**: The login name for the user (use lowercase, no spaces)
- **Required user actions**: Define what the user must do on first login (e.g., update password)
- **Email**: Assign a valid email address
- **Email verified**: If unchecked, the user will be prompted to verify their email on login
- **First / Last name**: Real names for identification
- **Credentials Tab**: Set the initial password
- **Groups Tab**: Only top-level groups are visible here — child groups must be assigned from the group view

---

## 4. Create the OIDC Client in Keycloak
To enable secure login from the Kubernetes Dashboard using Keycloak, we need to configure Keycloak as an OIDC identity provider. This is done by creating a client in Keycloak that represents the Kubernetes Dashboard as a trusted application.

### What is an OIDC Client?
An OIDC client in Keycloak defines:
- Who is allowed to authenticate (users/groups)
- Which application is requesting authentication (in this case, the Kubernetes Dashboard)
- How tokens are issued and validated
- Which claims (like groups, email, username) are included in the token

This client acts as the bridge between Keycloak and Kubernetes Dashboard, allowing users to log in using their Keycloak credentials and receive a token that Kubernetes can validate and use for RBAC.

### Setup
It's best for to setup a single client for each service. You could use the same but this might bring limitations we don't want.

#### General Settings
| Setting                                   | Value                                        | Description                      |
| :---                                      | :---                                         | :---   | 
| **Client type**                           | `OpenID Connect`                             | Protocol used for user authentication and token issuance |
| **ClientID**                              | `k8s-dashboard`                              | Unique identifier for the client application |
| **Name**                                  | `Kubernetes Dashboard`                       | Display name for the client in Keycloak UI |
| **Description**                           | `Kubernetes Dashboard OpenID Connect Client` | Optional description for clarity  |
| **Always display in UI**                  | `Off`                                        | Hides the client from the user's application list in the Keycloak account portal |

#### Capability config
| Setting                                   | Value                                        | Description                      |
| :---                                      | :---                                         | :---   | 
| **Client authentication**                 | `On`                                         | Enables client to authenticate using its secret during token exchange |
| **Authorization**                         | `Off`                                        | Disables Keycloak's internal authorization policies (RBAC handled in Kubernetes) |
| **Standard Flow**                         | `On`                                         | Enables the Authorization Code Flow for browser-based login|
| **Direct Access Grants**                  | `On`                                         | Allows token requests via username/password without browser interaction |
| **Implicit Flow**                         | `Off`                                        | Deprecated flow; disabled for security reasons |
| **Service accounts roles**                | `Off`                                        | Not needed; authentication is user-based, not service-based |
| **OAuth 2.0 Device Authorization Grant**  | `Off`                                        | Not applicable; used for devices without browsers  |
| **OIDC CIBA Grant**                       | `Off`                                        | Not applicable; used for decoupled login flows |

#### Access settings
| Setting                                   | Value                                        | Description                      |
| :---                                      | :---                                         | :---   | 
| **Root URL**                              | `https://kubedash.iot.keutgens.be`           | Base URL of the Kubernetes Dashboard application|
| **Home URL**                              | `https://kubedash.iot.keutgens.be`           | Landing page for the application |
| **Valid redirect URIs**                   | `https://kubedash.iot.keutgens.be/*`         | Allowed callback URLs after successful login  |
| **Valid post logout redirect URIs**       | `https://kubedash.iot.keutgens.be`           | URL to redirect users after logout |
| **Web origins**                           | `https://kubedash.iot.keutgens.be`           | Allowed origin for CORS requests during login and token exchange |

#### Login settings
| Setting                         | Value      | Description                      |
| :---                            | :---       | :---                             | 
| **Login theme**                 | empty      | Optional visual theme for the login screen; leave empty to use default  |
| **Consent required**            | Off        | Users are not prompted to approve access when logging in  |
| **Display client on screen**    | Off        | Hides the client from the user's application list in the Keycloak account portal  |
| **Client consent screen text**  | empty      | Optional custom text shown on the consent screen, not used in this setup  | 

#### Logout settings
| Setting                                         | Value      | Description                      |
| :---                                            | :---       | :---                             | 
| **Front channel logout**                        | On         | Enables browser-based logout via redirect from the application |
| **Front-channel logout URL**                    | empty      | Optional URL to trigger logout from the application; not required here |
| **Backchannel logout URL**                      | empty      | Optional server-side logout endpoint; not used in this setup |
| **Backchannel logout session required**         | On         | Ensures user sessions are properly terminated during backchannel logout |
| **Backchannel logout revoke offline sessions**  | Off        | Offline tokens remain valid after logout; no revocation applied |

--- 

## 5. Configure Kubernetes Dashboard with OIDC
To enable secure login to the Kubernetes Dashboard using Keycloak as an OpenID Connect (OIDC) identity provider, we need to configure the Dashboard to use OIDC and provide it with the necessary credentials. This involves creating a Kubernetes Secret to store the Keycloak client secret, and a Helm values file to configure the Dashboard with the correct OIDC parameters.

### Create the Kubernetes Secret
The Dashboard needs access to the Keycloak client secret to authenticate securely. You can find this secret in Keycloak under: 
Clients → k8s-dashboard → Credentials tab

Copy the value of the Client Secret and create the Kubernetes Secret manifest:
```bash
vi k8s-dashboard-secret.yaml
```
Paste the following content and replace <secret-here> with the actual secret:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-oidc-secret
  namespace: kubernetes-dashboard
type: Opaque
stringData:
  clientSecret: "<secret-here>"
```
Apply the secret to your cluster:
```bash
kubectl apply -f k8s-dashboard-secret.yaml
```

### Create the Dashboard Helm Values File
To configure the Dashboard with OIDC, create a Helm values file that includes the necessary authentication parameters:
```bash
vi k8s-dashboard-values.yaml
```
Add the following content:
```yaml
replicaCount: 2

affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app.kubernetes.io/name
              operator: In
              values:
                - kubernetes-dashboard
        topologyKey: "kubernetes.io/hostname"

oidc:
  enabled: true
  issuerUrl: https://keycloak.iot.keutgens.be/realms/iot-cluster
  clientId: k8s-dashboard
  clientSecret: "<your-keycloak-client-secret>"  # Replace or inject via secret
  groupsClaim: groups
  usernameClaim: preferred_username

extraArgs:
  - --authentication-mode=oidc
  - --enable-insecure-login=false
  - --token-ttl=1800
  - --token-cleanup=true

ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
  hosts:
    - host: kubedash.iot.keutgens.be
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - hosts:
        - kubedash.iot.keutgens.be
      secretName: kubedash-tls

metricsScraper:
  enabled: true

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

Deploy the Dashboard with OIDC:
Install or upgrade the Dashboard using Helm:
```bash
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --namespace kubernetes-dashboard \
  --values k8s-dashboard-values.yaml
```
















