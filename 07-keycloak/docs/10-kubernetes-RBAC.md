# Understanding Kubernetes RBAC & Keycloak
This setup secures access to the Kubernetes Dashboard using Keycloak as an identity provider via OIDC (OpenID Connect).

> This README is intended to provide a clear understanding of how Keycloak and Kubernetes work together using OIDC and RBAC. It does not contain actual configuration files or deployment manifests.

---

## Login Flow Overview
1. **User opens the Dashboard URL**: Example: `https://dashboard.iot.keutgens.be`
2. **Dashboard redirects to Keycloak**: The user is sent to Keycloak to authenticate via username/password or other login methods.
3. **Keycloak issues an OIDC token**: After successful login, Keycloak returns a JWT token containing user identity and group claims (e.g. groups: ["k8s-dashboard-viewers"]).
4. **Dashboard forwards the token to Kubernetes API**: The token is used to authenticate the user against the Kubernetes cluster.
5. **Kubernetes RBAC evaluates the token Kubernetes**: Checks the groups claim in the token and applies RBAC rules defined via ClusterRoleBinding.

---

## How Keycloak and Kubernetes RBAC Work Together
- **Keycloak handles authentication**: It verifies the user's identity and provides group membership via token claims.
- **Kubernetes handles authorization**: It uses RBAC to decide what each group can do (e.g. view-only, settings access, full admin).

---

## Roles and Bindings in Kubernetes
Kubernetes does not provide built-in roles like “admin” or “viewer” for the Dashboard. Instead, access is controlled via RBAC (Role-Based Access Control) using custom-defined roles and bindings.

### ClusterRole
A ClusterRole defines what actions are allowed on which Kubernetes resources. For example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: k8s-dashboard-viewer
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "deployments"]
    verbs: ["get", "list", "watch"]
```
This role `k8s-dashboard-viewer` allows read-only access to basic resources.

### ClusterRoleBinding
A ClusterRoleBinding connects a role to a group. The group name must match the groups claim in the Keycloak token:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: k8s-dashboard-viewer-binding
roleRef:
  kind: ClusterRole
  name: k8s-dashboard-viewer
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: Group
    name: k8s-dashboard-viewers
    apiGroup: rbac.authorization.k8s.io
```
This binding ensures that users in the `k8s-dashboard-viewers` group can view resources in the Dashboard.

---

## What You Secure with Keycloak and OIDC
This setup does not secure local access via SSH or kubectl using local kubeconfig. Instead, it secures web-based access to the Kubernetes Dashboard via OIDC (OpenID Connect). For example going to the webpage `https://dashboard.iot.keutgens.be` which runs Kubernetes-dashboard on my LAN-only network.

### What is secured:
- **Dashboard login**: Only users authenticated via Keycloak can access the UI.
- **Token-based access**: The JWT token issued by Keycloak contains group claims used for RBAC.
- **Granular permissions**: Each group is mapped to a specific role (e.g. viewer, settings, admin).
- **No impact on local access**: Users like iotadmin using local kubectl remain unaffected and retain full access.

This ensures that access to the Dashboard is tightly controlled, auditable, and aligned with your Keycloak group structure.

---

## RBAC Resources and Verbs Explained
In Kubernetes RBAC, access is defined by what resources a user can interact with, and which verbs (actions) they are allowed to perform. These are declared inside ClusterRole or Role objects.

### Common Resources
| Resource                  |	Description |
| :---                      | :---        | 
| pods	                    | Individual workloads (containers) running in the cluster |
| services	                | Network endpoints that expose pods |
| deployments	              | Declarative management of pod replicas |
| configmaps	              | Non-sensitive configuration data |
| secrets	                  | Sensitive data like passwords and tokens |
| namespaces	              | Logical separation of resources |
| persistentvolumeclaims	  | Requests for storage volumes |
| nodes	                    | Physical or virtual machines in the cluster |
| events	                  | System-level events and logs |
| ingresses	                | HTTP routing rules for external access |

### Common Verbs
| Verb              |	Description |
| :---              | :---        | 
| get	              | Retrieve a single resource |
| list	            | Retrieve all resources of a type |
| watch	            | Monitor changes to resources |
| create	          | Create a new resource |
| update	          | Modify an existing resource |
| patch	            | Apply partial changes to a resource |
| delete	          | Remove a resource |
| deletecollection	| Remove multiple resources at once |
| *	Wildcard:       | all verbs allowed |

## Example ClusterRoles
These examples define two access levels for the Kubernetes Dashboard, based on RBAC:

### Admin Role (Full Access)
This role grants full access to all resources and actions across the cluster:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: k8s-dashboard-admin
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
```
- apiGroups: `["*"]` : includes all API groups (core, apps, networking, etc.)
- resources: `["*"]` : includes all resource types (pods, services, secrets, etc.)
- verbs: `["*"]` : allows all actions (get, list, create, delete, etc.)
Use this only for trusted admin groups, such as k8s-dashboard-admins.

### Viewer Role (Read-Only Access)
```yaml
piVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: k8s-dashboard-viewer
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets", "persistentvolumeclaims"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses", "networkpolicies"]
    verbs: ["get", "list", "watch"]
```
- Covers common Dashboard-visible resources
- Only allows safe, read-only verbs: get, list, watch
- Does not allow create, update, or delete
Ideal for groups like k8s-dashboard-viewers who need observability without risk.


