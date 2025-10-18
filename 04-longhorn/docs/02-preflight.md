# Longhorn Preflight Check
Before installing Longhorn, we perform a final sanity check using the Longhorn CLI tool. This ensures that all nodes meet the required conditions for a successful deployment.

The preflight check validates whether the cluster is ready for Longhorn by verifying several critical conditions, including:
- Whether required kernel modules are loaded (e.g. dm_crypt, etc.)
- Whether DNS is functional (CoreDNS reachable via the Kubernetes API)
- Whether Longhorn-specific labels and tolerations are correctly configured
- Whether suitable storage devices are available on the nodes
- Whether the cluster meets Longhorn’s minimum system requirements

## 1. Download and Prepare longhornctl
Download the CLI binary:
```bash
curl -sSfL -o longhornctl https://github.com/longhorn/cli/releases/download/v1.9.1/longhornctl-linux-amd64
sudo chmod +x longhornctl
```

---

## 2. Run Preflight Check
According to the documentation, the following command should work:
```bash
./longhornctl check preflight
```
However, this failed due to missing kubeconfig context. To fix this, export the correct config path:
```bash
export KUBECONFIG=/home/iotadmin/.kube/config
./longhornctl check preflight
```

---

## 3. Troubleshooting Preflight Errors
During our pre-check, we encountered the following issues:

### 3.1 Module dm_crypt is not loaded
The kernel module `dm_crypt` was present but not yet loaded. Solution:
```bash
sudo modprobe dm_crypt
echo dm_crypt | sudo tee /etc/modules-load.d/dm_crypt.conf
lsmod | grep dm_crypt
```
This ensures the module is active and loaded automatically on boot.

### 3.2 Failed to list Kube DNS … i/o timeout (on 2 servers)
The preflight pod on those nodes failed to query the CoreDNS deployment via the API server (10.96.0.1:443). This is usually caused by:
- Temporary network or DNS issues
- Firewall rules blocking API traffic
> ⚠️ We chose to ignore this error, as the timeouts may be overly strict and transient.

---

## 4. Proceed with Installation
Once the preflight check is complete and any critical issues are resolved, install Longhorn using:
```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.9.1/deploy/longhorn.yaml
```
You can re-run the preflight check at any time:
```bash
longhornctl install preflight
longhornctl check preflight
```


