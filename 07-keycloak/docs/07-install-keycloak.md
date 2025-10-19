# Keycloak Deployment Configuration
This deployment launches the Keycloak container inside the `keycloak` namespace, with persistent storage, database connectivity, and health probes. It is designed for production-grade use with Ingress and TLS.

## ⚠️ Image Compatibility Notice
We are using an older Keycloak image version: `quay.io/keycloak/keycloak:22.0.5`

This version was chosen because it supports the --auto-build startup option, which is no longer available in newer Keycloak releases. The --auto-build flag allows Keycloak to automatically build its configuration from environment variables, simplifying container setup and avoiding manual configuration files.

