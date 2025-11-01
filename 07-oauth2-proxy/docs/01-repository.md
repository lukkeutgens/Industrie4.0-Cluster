# OAuth2-Proxy Helm Repository
To install OAuth2 Proxy in your Kubernetes cluster, we use the official Helm chart provided by the [OAuth2 Proxy manifests repository](https://github.com/oauth2-proxy/manifests) project. This Helm chart simplifies deployment and allows for easy configuration via values.yaml.

## Add the Helm Repository
```bash
helm repo add oauth2-proxy https://oauth2-proxy.github.io/manifests
helm repo update
```
This makes the latest version of the oauth2-proxy chart available for installation.


