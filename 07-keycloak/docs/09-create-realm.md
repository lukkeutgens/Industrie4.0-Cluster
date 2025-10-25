# Setup Keycloak Realm
Login on the Keycloak Administration Console.
In my case this is on `https://keycloak.iot.keutgens.be/` (only running in my LAN).
You will see the following items:

## Manage Section
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

## 1. Create a new Realm
When you first login, you will be in the `master` Realm. When you click on that name there is the option to make a new one, so let's do this.
- **Resource file** : To import an earlier made Realm, not needed in our case
- **Realm name** : The name for our new Realm. In my case this will be `iotcluster`
- **Enabled** : Needs to `On` off course.
Click on `Create`

---

## 2. Edit Realm Settings
After creating the Realm we can further edit the settings to make it more complete.

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

Our settings:
- **Realm ID** : iotcluster
- **Display Name** : IoT-Cluster
- **HTML Display name** : <strong>IoT Cluster Login</strong>
- **Frontend URL** : https://keycloak.iot.keutgens.be
- **Require SSL** : All requests
The rest we will live unchanged.






