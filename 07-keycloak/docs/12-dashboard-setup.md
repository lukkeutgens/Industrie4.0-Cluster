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
Now I create 3x users. One as admin, the other for the settings and the last one as viewer only.
> When using `Child Groups` you will have to go to that group itself to assign users to it. It's not possible in the **User edit** page to assing that user to a child group, there you can only select the Main Groups.

| User      | User Group                |   Roles (auto from group)  |
| :---      | :---                      | :---        |
| iotadmin  | `k8s-dashboard/admins`    | `k8s-admin`  |
| luk       | `k8s-dashboard/settings`  | `k8s-viewer`, `k8s-settings` |
| leen      | `k8s-dashboard/viewers`   | `k8s-viewer`  | 

Settings for a user:
- **Username**: The login name for that user (no capital letters)
- **Required user actions**: Select what a user should do when he logs in the first time.
- **Email**: Give the user an email
- **Email verified**: If not then the user needs to verify it when he logs in
- **First name**: Real first name for user
- **Last name**: Real last name for user
- **Credentials**: In this tab you can setup a password
- **Groups**: In this tab you can assign main groups, not child groups. We have used child groups so we can not add the user to the groups here.



- 





