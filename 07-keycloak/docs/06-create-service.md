# Keycloak Service Definition
This Kubernetes Service exposes the Keycloak pod internally within the cluster. It enables other components (such as Ingress, monitoring tools, or other services) to reach Keycloak via a stable DNS name and port.

The service is of type `ClusterIP`, meaning it is only accessible within the cluster. It forwards traffic on port `8080` to the Keycloak container listening on the same port.

## 1. Create service file
Create file:
```bash
vi keycloak-service.yaml
```
Add content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: keycloak-service
  namespace: keycloak
  labels:
    app: keycloak
spec:
  type: ClusterIP
  selector:
    app: keycloak
  ports:
    - name: http
      port: 8080
      targetPort: 8080
```
- The selector matches pods with the label app=keycloak, ensuring traffic is routed to the correct container.
- The service is used by the Ingress resource to forward external HTTPS traffic to the internal Keycloak pod.

## 2. Apply the service
```bash
kubectl apply -f keycloak-service.yaml
```
Once applied, the service will be available at:
```bash
keycloak-service.keycloak.svc.cluster.local:8080
```
This endpoint is used by the Ingress and can also be referenced by other services or test pods inside the cluster.


