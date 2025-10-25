# Creating Users and Roles in Keycloak
This guide explains how to create and configure users in the `iotcluster` realm to authenticate with the Kubernetes Dashboard via Keycloak and OIDC. It includes an admin user (`iotadmin`), a technical user (`luk`), and a read-only user (`leen`).

---

## 1. Realm Roles
We need to create Realm Roles that we can link with the user groups. These roles will define what a user can do or not do.

### Default Keycloak Roles
| Role Name      | Description            | Recomended Use    |
| :---           | :---                   | :---              |
| default-roles-iot-cluster | This role serves as a container for both realm and client default roles. It cannot be removed. (default Keycloak) |   |
| offline_access  | Allows clients to request refresh tokens that remain valid when the user is offline. Useful for background tasks or long-lived sessions. (default Keycloak) | Recommended for Kubernetes Dashboard if you want persistent sessions without re-login. |
| uma_authorization | Enables the User-Managed Access (UMA) protocol. Lets users share access to their resources via OAuth2. (default Keycloak) | Rarely needed in standard cluster setups. Disable unless explicitly required. | 

### Custom Created Roles
| Role Name      | Description                           |
| :---           | :---                                  |
| k8s-admin      | Full access to Kubernetes (Dashboard) |
| k8s-settings   | Limited access to dashboard settings  |
| k8s-viewer     | Read-only access to Kubernetes Dashboard |
| longhorn-admin  | Full control over Longhorn volumes and settings  |
| longhorn-settings  | Access to Longhorn settings, no destructive actions |
| longhorn-viewer | Read-only access to Longhorn UI | 
| keycloak-admin | Full realm administration (can be replaced with built-in admin role) |
| keycloak-usermanager | Manage users only, no access to groups or realm settings |
| keycloak-viewer | View-only access to users and realm settings |

---

## User Group
Let's create the user groups



---

## User Overview
| Username     | Display Name   | Role in Keycloak | Role in Kubernetes Dashboard | 2FA   | Notes                 |
| :---         | :---           | :---             | :---                         | :---  | :---         |
| `iotadmin`   | –              | Realm Admin      | Cluster Admin                | ✅    | Created via Kubernetes secret |
| `luk`        | Luk Keutgens   | None             | Cluster Admin                | ✅    | No access to Keycloak admin |
| `leen`       | Leen Sels      | None             | Read-only                    | ✅    |  Read-only  |

---

## Create a user
Field	Description	Recommended Input:
| Field                  | Input            | Description       |
| :---                   | :---             | :---              |
| Required User Actions  | empty for now    | Optional dropdown to enforce actions on next login (e.g., update password, configure OTP). |
| Username               | `iotadmin`, `luk`, `leen`, no spaces or special characters.        | Unique identifier for the user. Used for login and API access. |
| Email                  | enter address | User’s email address. Used for notifications and account recovery. |
| Email Verified  | `ON` if trusted  | Marks the email as verified. Required for some flows like password reset | 
| First Name  | Ex. `Luk`  | User’s first name. Displayed in UI and tokens. |
| Last Name    | Ex. `Keutgens` | User’s last name. Displayed in UI and tokens. | 
| Groups      | created group  | Assigns the user to predefined groups with shared roles or permissions. | 

