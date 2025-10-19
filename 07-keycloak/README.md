# Keycloak Central Identity and Access Management (IAM)
This document describes the setup for integrating Keycloak as the central Identity and Access Management (IAM) solution within the Kubernetes cluster. Keycloak will be deployed as a containerized service and connected to the existing PostgreSQL database, which is already running as a StatefulSet backed by Longhorn volumes.

> ⚠️ Image Compatibility Notice We are using Keycloak version 22.0.5 because it supports the --auto-build flag. Newer versions require a separate build step, which failed during testing. See install-keycloak.md for details.

---

## Why Keycloak?
Keycloak is a robust, open-source IAM platform that supports modern authentication protocols and integrates seamlessly with containerized environments. It is well-suited for securing industrial services in a scalable and maintainable way.

### Benefits
- Container-native: Runs reliably in Kubernetes using the official container image from quay.io/keycloak/keycloak.
- Strong MFA/2FA support: Built-in support for OTP, hardware tokens, and future integration with PrivacyIDEA.
- Protocol support: Full compatibility with OAuth2, OpenID Connect, and SAML.
- Central IAM: Acts as a unified login provider for all services in the cluster.
- Extensible: Can be customized with themes, providers, and external identity sources (LDAP, AD).

### Target Integrations
Keycloak will be used to secure access to:
- Kubernetes Dashboard (via OpenID Connect, if compatible)
- Longhorn UI (via OAuth2 or reverse proxy integration, if supported)

> ⚠️ These integrations will be evaluated and documented in follow-up steps.

---

## Deployment Strategy
- Database: Keycloak will connect to the existing PostgreSQL instance (timescaledb-0.postgresql.svc.cluster.local:5432) using environment variables and Kubernetes secrets.
- Container Image: We will use the official image quay.io/keycloak/keycloak:latest.
- Build Mode: Keycloak requires a build step before startup. This will be handled via an init container or pre-built image using kc.sh build.
- Namespace: Keycloak will be deployed in its own namespace (keycloak) with dedicated PVCs and secrets.
- Node Placement: Keycloak will run with two replicas, scheduled explicitly on nodes csrv1 and wsrv1 using nodeAffinity rules. This ensures predictable placement and performance.
- Persistent Storage: A dedicated Longhorn volume will be manually provisioned with two replicas, each pinned to csrv1 and wsrv1. This guarantees high availability and data locality for Keycloak's persistent state.

---

## References
- [Keycloak website](https://www.keycloak.org/)
- [Container image](https://quay.io/repository/keycloak/keycloak)
- [Container docs](https://www.keycloak.org/server/containers)

---

## Folder Content
```text
├── docs
│   ├── 01-create-volume.md        # Create PVC and PV via Longhorn UI for persistent Keycloak data
│   ├── 02-create-database.md      # PostgreSQL setup: create DB and user for Keycloak
│   ├── 03-create-secret.md        # Define DB credentials as Kubernetes Secret for Keycloak
│   ├── 04-create-cert.md          # Generate TLS certificate via cert-manager for HTTPS
│   ├── 05-create-ingress.md       # Configure Ingress to expose Keycloak over HTTPS
│   ├── 06-create-service.md       # Internal ClusterIP service to route traffic to Keycloak pod
│   ├── 07-install-keycloak.md     # Deploy Keycloak container with auto-build and DB integration
│   └── 08-debugging-help.md       # Troubleshooting help
├── files
│   ├── keycloak-cert.yaml         # TLS certificate resource
│   ├── keycloak-deploy.yaml       # Keycloak Deployment
│   ├── keycloak-ingress.yaml      # Ingress resource
│   ├── keycloak-pvc.yaml          # PersistentVolumeClaim
│   ├── keycloak-pv.yaml           # PersistentVolume definition (Longhorn-backed)
│   ├── keycloak-secret.yaml       # Secret containing DB credentials for Keycloak
│   └── keycloak-service.yaml      # ClusterIP service
└── README.md                      # This file
```
