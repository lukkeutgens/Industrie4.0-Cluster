# Keycloak Ingress Configuration
This Ingress resource exposes Keycloak over HTTPS using NGINX Ingress Controller already active in the cluster. It routes traffic for `keycloak.iot.keutgens.be` to the internal Keycloak service.

## 1. Create the Ingress file
Create the file: 
```bash
vi keycloak-ingress.yaml
```
Add content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: keycloak-ingress
  namespace: keycloak
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "128k"
spec:
  tls:
    - hosts:
        - keycloak.iot.keutgens.be
      secretName: keycloak-tls
  rules:
    - host: keycloak.iot.keutgens.be
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: keycloak-service
                port:
                  number: 8080
```
- TLS is enabled via the keycloak-tls secret, which must be created by cert-manager (see keycloak-cert.yaml).
- The backend service keycloak-service must be running and reachable on port 8080.

## 2. Apply the Ingress:
```bash
kubectl apply -f keycloak-ingress.yaml
```
Once applied, traffic to https://keycloak.iot.keutgens.be will be routed to your Keycloak deployment.


