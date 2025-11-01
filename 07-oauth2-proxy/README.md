# OAuth2-Proxy Authentication Proxy
OAuth2 Proxy is a lightweight authentication proxy that sits in front of web applications and enforces access control using OAuth2 or OpenID Connect (OIDC) identity providers. It acts as a gatekeeper: users must authenticate via a trusted provider (e.g., Keycloak) before gaining access to the protected service.

Links:
- Docs: https://oauth2-proxy.github.io/oauth2-proxy/
- Github: https://github.com/oauth2-proxy/oauth2-proxy
- Helm Manifests: https://github.com/oauth2-proxy/manifests

## Why OAuth2 Proxy?
Most Kubernetes-native services—like `Kubernetes Dashboard` do not natively support external identity providers such as Keycloak. OAuth2 Proxy bridges this gap by:
- Handling authentication flows with Keycloak (OIDC)
- Injecting identity headers (e.g., X-Auth-Request-Email, Authorization) into upstream requests
- Protecting services behind a reverse proxy with fine-grained access control
- Supporting cookie-based sessions for user convenience

## Use Case: Kubernetes Dashboard + Keycloak
The Kubernetes Dashboard does not support OIDC out of the box. By placing OAuth2 Proxy in front of it, we can:
- Authenticate users via Keycloak (e.g., using realm roles or groups)
- Restrict access based on identity attributes
- Maintain session security with signed cookies
- Avoid exposing the dashboard to unauthenticated traffic

## How It Works
1. A user accesses the dashboard via an Ingress URL (e.g., https://kubedash.iot.keutgens.be)
2. OAuth2 Proxy intercepts the request and redirects to Keycloak for login
3. Upon successful authentication, OAuth2 Proxy sets a secure cookie and forwards the request to the dashboard
4. The dashboard receives identity headers and can be configured to trust them

This setup enables centralized identity management across multiple services in the cluster, using Keycloak as the single source of truth.

