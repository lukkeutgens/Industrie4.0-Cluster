# Creating Users and Roles in Keycloak
This guide explains how to create and configure users in the `iotcluster` realm to authenticate with the Kubernetes Dashboard via Keycloak and OIDC. It includes an admin user (`iotadmin`), a technical user (`luk`), and a read-only user (`leen`).

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

