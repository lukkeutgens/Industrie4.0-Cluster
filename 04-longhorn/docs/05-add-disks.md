# Adding Data Disks to Longhorn
Now we need to add our data disks to Longhorn so they can be used for volume storage.

| Field             | Meaning                             | Recommended for  
| :---              | :---                                | :---
| Disk type         | How Longhorn accesses the disk      | File System (you already have a mounted path like /data)
| Storage Available | Free space Longhorn may use         | Leave empty → Longhorn calculates automatically
| Storage Scheduled | Space already allocated to volumes  | Auto-filled by Longhorn
| Storage Max       | Max space Longhorn may use          | Leave empty unless you want to limit (e.g. 50Gi)
| Storage Reserved  | Space to exclude from Longhorn usage | e.g. 5Gi if /data is shared with other services
| Scheduling        | Whether Longhorn can schedule volumes here | Enable (checked)
| Eviction Requested | Whether to evict all volumes from this disk | Unchecked (unless you want to drain the disk)

---

## View Disks on Debian
To list available disks and mount points:
```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,MODEL
```
This gives a clear overview of your block devices and mount paths.

---

## Add Disks via the Longhorn UI
Navigate to:
Node → Select node → Edit node & disks → Add disk

Use the following values:
- Disk type: File System
- Path: /data
- Disk name: data-disk
- Scheduling: enabled
- Eviction Requested: disabled
- Storage Reserved: 5Gi (optional, to reserve space)
- Storage Max: leave empty (Longhorn will use all available space minus reserved)
- Storage Available / Scheduled: auto-filled by Longhorn

---

## Add Disks via kubectl (Control Plane)
List available Longhorn nodes:
```bash
kubectl get nodes.longhorn.io -n longhorn-system
```
Edit the desired node (e.g. wsrv2):
```bash
kubectl edit nodes.longhorn.io wsrv2 -n longhorn-system
```
Add the new disk under spec.disks:
```yaml
spec:
  allowScheduling: true
  disks:
    default-disk-4d1da2023cf680fc:
      allowScheduling: true
      diskDriver: ""
      diskType: filesystem
      evictionRequested: false
      path: /var/lib/longhorn/
      storageReserved: 14666242867
      tags: []
    disk-data:
      allowScheduling: true
      diskDriver: ""
      diskType: filesystem
      evictionRequested: false
      path: /data
      storageReserved: 5368709120
      tags: []
```
> Tip: storageReserved is in bytes. 5368709120 = 5Gi.

---

## Exporting disks to YAML file
Export full YAML from a node:
```bash
kubectl get nodes.longhorn.io <nodename> -n longhorn-system -o yaml
```
Example export to file for backup
```bash
kubectl get nodes.longhorn.io csrv1 -n longhorn-system -o yaml > csrv1-disks.yaml
```

