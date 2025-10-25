# Kubernetes-Dashboard Login Setup
> Please read also `10-kubernetes-RBAC` to get a better understanding of what we are going to setup!

In this document I will be using Keycloak to create a secure login with OpenID Connect (OIDC) to login to Kuberetes-Dashboard.

I will create the Roles, Groups and Users that will be allowed to login on the dashboard with different rights.

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
Now we will create the user groups that can access the Kubernetes-dashboard and add the roles to them. We will use the group "k8s-dashboard" and create 3 child groups as set below:

| User Group      | Child Group | Roles       |
| :---            | :---        | :---        |
| k8s-dasbboard   | viewers     | `k8s-viewer`  | 
| k8s-dashboard   | settings    | `k8s-viewer`, `k8s-settings` |
| k8s-dashboard   | admins      | `k8s-admin` |


