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
| k8s-dasbboard   | viewers     | `k8s-viewer`  | 
| k8s-dashboard   | settings    | `k8s-viewer`, `k8s-settings` |
| k8s-dashboard   | admins      | `k8s-admin` |

Each child group receives the appropriate Keycloak roles. These roles determine what the user can see or manage in the dashboard.

To match Keycloak groups in Kubernetes RBAC, use the full group path as seen in the JWT token. For example:
```yaml
subjects:
  - kind: Group
    name: /k8s-dasbboard/viewer
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

So for our Kubernetes Dashboard:
- **Client type**: OpenID Connect
- **ClientID**: k8s-dashboard     <- Unique name for client
- **Name**: Kubernetes Dashboard  <- Usefull name
- **Description**: Kubernetes Dashboard OpenID Connect Client
- **Always display in UI**: Off (if client is shown in users account UI)
- **Client authentication**: On
- **Authorization**: Off (is for Keycloak internal authorization services)
- **Authentication Flows**: 
    - **Standard Flow**: On (needed for browser based login)
    - **Implicit Flow**: Off (older)
    - **Direct Access Grants**: On (For flexibility)
    - **OAuth 2.0 Device Authorization Grant: Off (for devices without browser)
    - **OIDC CIBA Grant**: Off (greyed out, flow for decoupled login, not for us)
 
- 


