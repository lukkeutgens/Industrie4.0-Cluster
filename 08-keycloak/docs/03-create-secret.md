# Keycloak Kubernetes Secret
We create a Kubernetes Secret containing both the database credentials for Keycloak and the initial admin login for the Keycloak service itself.

Create the file:
```bash
vi keycloak-secret.yaml
```
Add the following content (adjust values as needed):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keycloak-secret
  namespace: keycloak
type: Opaque
stringData:
  KC_DB_USERNAME: keycloak_user
  KC_DB_PASSWORD: ********                # Replace with actual password
  KEYCLOAK_ADMIN: iotadmin
  KEYCLOAK_ADMIN_PASSWORD: *********      # Replace with actual password
```
Apply the secret to your cluster:
```bash
kubectl apply -f keycloak-secret.yaml
```

The secret keycloak-secret is now available in the keycloak namespace and can be referenced in your Keycloak Deployment to inject environment variables for database access and admin setup.
