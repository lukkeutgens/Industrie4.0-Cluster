# Creating Users and Roles in Keycloak
This guide explains how to create and configure users in the `iotcluster` realm to authenticate with the Kubernetes Dashboard via Keycloak and OIDC. It includes an admin user (`iotadmin`), a technical user (`luk`), and a read-only user (`leen`).

---

## User Overview
| Username     | Display Name   | Role in Keycloak | Role in Kubernetes Dashboard | 2FA   | Notes                 |
| :---         | :---           | :---             | :---                         | :---  | :---         |
| `iotadmin`   | –              | Realm Admin      | Cluster Admin                | ✅    | Created via Kubernetes secret |
| `luk`        | Luk Keutgens   | None             | Cluster Admin                | ✅    | No access to Keycloak admin |
| `leen`       | Leen Sels      | None             | Read-only                    | ✅    |  Read-only  |
