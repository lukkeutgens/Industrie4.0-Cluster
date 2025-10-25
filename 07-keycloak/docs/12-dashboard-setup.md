# Kubernetes-Dashboard Login Setup
> Please read also `10-kubernetes-RBAC` to get a better understanding of what we are going to setup!

In this document I will be using Keycloak to create a secure login with OpenID Connect (OIDC) to login to Kuberetes-Dashboard.

I will create the Roles, Groups and Users that will be allowed to login on the dashboard with different rights.

---

## 1. Create Keycloak Realm Roles
Viewer role:
- **Role name** : k8s-viewer
- **Description** : Read-only access to Kubernetes Dashboard
- Attributes:
| key        | Value              |
| :---       | :---               |
| apiGroups  | ["", "apps", "batch", "networking.k8s.io"]  |
| resources  | ["pods", "services", "events", "deployments", "replicasets", "statefulsets", "jobs", "cronjobs", "ingresses"]  |
| verbs      | ["get", "list", "watch"]  |
