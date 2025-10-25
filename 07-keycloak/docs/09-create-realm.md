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
