# Installing Longhorn via kubectl
Now we install Longhorn itself using the official deployment manifest:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.9.1/deploy/longhorn.yaml
```

## Common Warnings (Not Critical)
During installation, you may see warnings like:
- missing ... last-applied-configuration annotation → This simply means the namespace already existed or was created earlier without that metadata. kubectl apply adds it automatically.
- unrecognized format "int64" / "int32" → Cosmetic warnings from kubectl while parsing the CRD YAML. The Custom Resource Definitions are still created correctly.

These are not show-stoppers and can be safely ignored.

## Verify Installation
To monitor the deployment and ensure all pods are running:
```bash
kubectl get pods -n longhorn-system --watch
```

> ℹ️ Note: Technically, the installation was already triggered during the preflight phase — but this step ensures the full Longhorn stack is deployed and active.

