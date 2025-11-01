# Keycloak Deployment Debug Help
This guide documents my debugging steps for validating and restarting the Keycloak deployment, including readiness probe testing, PVC detachment, and live pod inspection.

---

## Readiness Probe Validation
Keycloak exposes health endpoints that Kubernetes uses to determine pod readiness. To manually validate the readiness probe.

Launch a test pod:
```bash
kubectl run test-client --rm -i --tty --image=curlimages/curl -n keycloak -- sh
```
Test the health endpoint:
```bash
curl -v -H "Host: keycloak.iot.keutgens.be" http://keycloak-service:8080/health/ready
```
Expected output:
If the endpoint returns 200 OK, the readiness probe is correctly configured.

---

## Clean deployment Restart
When modifying the deployment (e.g. volume mounts, probes, image), it's important to fully remove the existing pod to ensure the PersistentVolume is properly detached. Maybe it's because I'm using VirtualBox on Windows 11, but I had issues with pods terminating correctly.

Delete the deployment:
```bash
kubectl delete deployment keycloak -n keycloak
```
Wait for pod termination:
```bash
kubectl get pods -n keycloak -w
```
Ensure all pods are terminated before reapplying the deployment.

Reapply the updated deployment:
```bash
kubectl apply -f keycloak-deploy.yaml
```

---

## Live Pod Inspection
Keycloak startup can take several minutes. Use the following commands to monitor progress.

First get the actual pod-name:
```bash
kubectl get pods -n keycloak
````

View startup logs:
```bash
kubectl logs <pod-name> -n keycloak -f
```

Describe the pod:
```bash
kubectl describe pod <pod-name> -n keycloak
```
Replace <pod-name> with the actual pod name (use kubectl get pods -n keycloak to find it)

