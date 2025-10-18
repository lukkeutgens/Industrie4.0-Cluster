# Create Longhorn Volume
We will create a Longhorn volume manually through the UI, ensuring it is placed on the correct nodes and disks using tags. After that, we will register this volume in Kubernetes using a PersistentVolume manifest.

---

## Longhorn Setup
Make the following changes through the Longhorn-UI

1. Under **Nodes**, select the nodes you want to use and assign a **Node tag**: `PostgreSQL`.
2. For each selected node, choose the disk you want to use and assign a **Disk tag**: `PostgreSQL`.
3. Go to **Volumes** and create a new volume using the settings below

### Volume Configuration

| Setting                  | Value            | Description                                                       |
| :---                     | :---             | :---                                                              |
| **Name**                 | `postgre-data`   | Unique name for the volume                                        |
| **Size**                 | `8Gi`            | Logical size of the volume                                        |
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
| **Disk tag**             | `PostgreSQL`     | Ensures volume is placed only on disks with this tag              |

Once created, the volume will appear in Longhorn with status `Detached`. This is expected — Kubernetes will attach it automatically when the pod is scheduled.

---

## Kubernetes Setup
### 1. Create the namespace
First we create the namespace `postgresql` we are going to use for PostgreSQL database.
```bash
kubectl create namespace postgresql
```

### 2. Create PersistentVolume (PV)
Create the file `pg-timescale-pv.yaml`:
```bash
vi pg-timescale-pv.yaml
```
Paste the following content:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgre-data
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  storageClassName: longhorn
  csi:
    driver: driver.longhorn.io
    volumeHandle: postgre-data
```
Apply the manifest:
```bash
kubectl apply -f pg-timescale-pv.yaml
```

### 3. Create PersistentVolumeClaim (PVC)
Create the file `pg-timescale-pvc.yaml`
```bash
vi pg-timescale-pvc.yaml
```
Paste the following content:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pg-timescale-pvc
  namespace: postgresql
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: longhorn
  volumeName: postgre-data
```
Apply the manifest:
```bash
kubectl apply -f pg-timescale-pvc.yaml
```

At this point, the PVC should bind to the manually registered PV, and Kubernetes will attach the volume when the pod is scheduled.



