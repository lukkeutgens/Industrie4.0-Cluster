# Keycloak Service Definition
This Kubernetes Service exposes the Keycloak pod internally within the cluster. It enables other components (such as Ingress, monitoring tools, or other services) to reach Keycloak via a stable DNS name and port.

The service is of type `ClusterIP`, meaning it is only accessible within the cluster. It forwards traffic on port `8080` to the Keycloak container listening on the same port.
