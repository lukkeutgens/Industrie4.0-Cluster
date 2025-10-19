# Create Longhorn Volume
We will create a Longhorn volume manually through the UI, ensuring it is placed on the correct nodes and disks using tags. After that, we will register this volume in Kubernetes using a PersistentVolume manifest.

---

## Longhorn Setup
Configure the following settings in the Longhorn UI:

1. Under `Nodes`, select the nodes you want to use and assign a **Node tag**: `keycloak`.
2. For each selected node, choose the disk you want to use and assign a **Disk tag**: `keycloak`.
3. Go to **Volumes** and create a new volume using the settings below

### Volume Configuration

| Setting                  | Value            | Description                                                       |
| :---                     | :---             | :---                                                              |
| **Name**                 | `keycloak-data`   | Unique name for the volume                                        |
| **Size**                 | `5Gi`            | Logical size of the volume                                        |
| **Number of replicas**   | `2`              | Number of data replicas across nodes for high availability        |
| **Data engine**          | `v1`             | Default engine; supports snapshots and backups                    |
| **Frontend**             | `Block device`   | Exposes volume as a block device to the pod                       |
| **Data locality**        | `Disabled`       | No forced node affinity; scheduler decides best placement         |
| **Access mode**          | `ReadWriteOnce`  | Volume can be mounted by a single node at a time                  |
| **Backing Image**        | n.a.             | Leave empty unless using a preloaded image                        |
| **Data source**          | n.a.             | Leave empty unless restoring from another volume or backup        |
| **Backup target**        | `default`        | Default backup location (can be configured in Longhorn settings)  |
| **Encrypted**            | n.a.             | Leave empty unless encryption is enabled                          |
| **Node tag**             | n.a.             | Not used directly; placement is controlled via Disk tag           |
| **Disk tag**             | `keycloak`     | Ensures volume is placed only on disks with this tag              |

Once created, the volume will appear in Longhorn with status `Detached`. This is expected — Kubernetes will attach it automatically when the pod is scheduled.

---

## Kubernetes Setup
### 1. Create the namespace
First we create the namespace `keycloak` we are going to use for Keycloak AIM.
```bash
kubectl create namespace keycloak
```
### 2. Create PersistentVolume (PV)
Create the file `keycloak-pv.yaml`:
```bash
vi keycloak-pv.yaml
```
Paste the following content:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: keycloak-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  storageClassName: longhorn
  csi:
    driver: driver.longhorn.io
    volumeHandle: keycloak-data
```
Apply the manifest:
```bash
kubectl apply -f keycloak-pv.yaml
```

### 3. Create PersistentVolumeClaim (PVC)
Create the file `keycloak-pvc.yaml`
```bash
vi keycloak-pvc.yaml
```
Paste the following content:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: keycloak-pvc
  namespace: keycloak
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
  storageClassName: longhorn
```
Apply the manifest:
```bash
kubectl apply -f keycloak-pvc.yaml
```

At this point, the PVC should bind to the manually registered PV, and Kubernetes will attach the volume when the pod is scheduled. In the longhorn-UI it is still shown as `Detached` but this will change when a pod claims the volume.

To check the PV and PVC are setup correctly use the following commands:
```bash
kubectl get pv keycloak-pv
kubectl get pvc keycloak-pvc -n keycloak
```
You should see the `bound` status on both.

