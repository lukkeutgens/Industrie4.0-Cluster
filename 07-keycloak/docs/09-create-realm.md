# Creating a Keycloak Realm for IoT Cluster
This guide explains how to manually create a new Keycloak realm for your Industrie4.0 Kubernetes cluster. The realm will be used to authenticate users for services like the Kubernetes Dashboard via OIDC and 2FA.

## Accessing the Keycloak Admin Console
Log in to the Keycloak Administration Console. In our setup, this is available at: `https://keycloak.iot.keutgens.be` (only LAN in my case)

| Section         | Description              | 
| :---            | :---                     |
| Master (Realm)  | Default realm used to manage Keycloak itself. Contains admin users and system-level configuration. |
| Clients         | Applications that use Keycloak for authentication (e.g., Kubernetes Dashboard). Configure OIDC/SAML settings here. |
| Client Scopes   | Collections of claim mappers and protocol settings that can be assigned to clients (e.g., profile, email). |
| Realm Roles     | Role definitions within a realm. Can be assigned to users or groups (e.g., admin, viewer). |
| Users           | User accounts within the realm. Configure credentials, roles, and two-factor authentication (2FA). |
| Groups          | Logical grouping of users for shared role assignments or attributes. |
| Sessions        | Active login sessions. Useful for monitoring and troubleshooting.  |
| Events          | Audit logs for login attempts, errors, and system actions. Can be forwarded to external systems. |

## Configuration Section
| Section             | Description              | 
| :---                | :---                     |
| Realm Settings      | Global realm configuration: login policies, token lifespans, themes, SMTP, etc. |
| Authentication      | Login flows, registration, and 2FA setup. Configure TOTP and custom login screens. |
| Identity Providers  | External login sources like Google, GitHub, or other Keycloak servers. |
| User Federation     | Integration with external user directories such as LDAP or Active Directory. | 

---

## 1. Creating a New Realm
After logging in, you will start in the `master` realm. To create a new realm:
1. Click the realm name in the top-left corner.
2. Select **Create Realm**.

You will see the following options:

- **Resource file**: Optional. Used to import a previously exported realm configuration. Leave empty for manual setup.
- **Realm name**: Set to `iotcluster`
- **Enabled**: Toggle to `On`
Click **Create** to proceed.

---

## 2. Realm Settings Configuration
Once the realm is created, configure its settings to match the cluster architecture and security requirements.

| Setting               | Description              | 
| :---                  | :---                     |
| Realm ID              | Internal identifier for the realm (e.g., iotcluster). Used in URLs and API calls. |
| Display Name          | Optional name shown in the Admin Console UI. Can be more descriptive than the ID. |
| HTML Display Name     | Optional HTML-formatted name shown in login screens. Supports basic HTML tags. |
| Frontend URL          | Overrides the default frontend URL used in redirects. Useful behind reverse proxies or Ingress. |
| Require SSL           | nforces HTTPS for all requests. Options: none, external requests, or all requests. |
| ACR to LoA Mapping    | Maps Authentication Context Class References (ACR) to Levels of Assurance (LoA). Used in advanced SSO setups. |
| User-Managed Access   | Enables UMA protocol for user-controlled resource sharing. Rarely used unless integrating with OAuth2 resource servers. | 
| Endpoints             | Shows all available endpoints for the realm (e.g., token, userinfo, certs). Useful for debugging and integration. |

### Recommended Settings
- **Realm ID** : iotcluster
- **Display Name** : IoT-Cluster
- **HTML Display name** : <strong>IoT Cluster Login</strong>
- **Frontend URL** : https://keycloak.iot.keutgens.be
- **Require SSL** : All requests

Leave other settings unchanged unless you have specific requirements.







