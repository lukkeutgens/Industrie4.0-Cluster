# Cluster Architecture
This document outlines the current architecture of my Industry 4.0 test environment. The setup is still in progress, but this serves as a snapshot of where things stand and how the system is being designed.

---

## Host Platform
- **Laptop OS**: Windows 11 Home (AMD Ryzen 9 7945HX3D with Radeon Graphics 32GB RAM)
- **Virtualization**: Oracle VirtualBox
- **Reasoning**: Minimal impact on host system, despite occasional instability due to nested virtualization conflicts with Hyper-V.

---

## Virtual Machines
All VMs run **Debian 12 (headless)** and are configured with fixed IPs and basic OS hardening.

| Hostname | Role           | IP Address       | CPU | RAM | Disk 1 (sda) | Disk 2 (sdb) |
|----------|----------------|------------------|-----|-----|--------------|--------------|
| csrv1    | Control Plane  | 192.168.50.160   |  6  | 6GB | 60GB         | 140GB        |
| wsrv1    | Worker Node 1  | 192.168.50.161   |  4  | 4GB | 60GB         | 140GB        |
| wsrv2    | Worker Node 2  | 192.168.50.162   |  4  | 4GB | 60GB         | 140GB        |
| wsrv3    | Worker Node 3  | 192.168.50.163   |  4  | 4GB | 60GB         | 140GB        |

---

## Networking

- **Domain**: `iot.keutgens.be` (LAN-only, not publicly resolvable)
- **IP assignment**: Static via `/etc/network/interfaces`
- **Hostnames**: Set per node, see [Virtual Machines](#virtual-machines)
- **Kubernetes networking**:
    - dnsDomain: cluster.local
    - podSubnet: 192.168.0.0/16 - Assigned by CNI plugin (Flannel, via /etc/cni/net.d/)
    - serviceSubnet: 10.96.0.0/12 - Internal service IPs (ClusterIP range)
    - DNS: CoreDNS handles internal resolution via cluster.local. Services are reachable via <service>.<namespace>.svc.cluster.local.
- **Ingress**: External access through NGINX Ingress Controller
    - Service type: LoadBalancer
    - External IP: 192.168.50.210 (static reserved)
    - ClusterIP: 10.111.52.55 (cluster internal)
    - Ports: 80 (HTTP), 443 (HTTPS)
    - Services  
        - Kubernetes-dashboard: 192.168.50.210 - kubedash.iot.keutgens.be (80/443) 
        - Longhorn-UI: 192.168.50.210 - longhorn.iot.keutgens.be (80/443)
- **LoadBalancer IP-pool** (MetalLB in Layer2-mode)
    - IP range: 192.168.50.210 - 192.168.50.220
    - Pool name: lb-pool
    - Protocol: Layer2 (ARP) 
---

## Disk Layout (per node)
```text
NAME                 SIZE FSTYPE      MOUNTPOINT          LABEL
sda                   60G
└─sda1                60G LVM2_member
  ├─LV_sys-LV_root  46,6G ext4        /                   rootfs
  └─LV_sys-LV_swap   1,9G swap
sdb                  140G
└─sdb1               140G LVM2_member
  ├─LV_data-LV_k8s  37,3G xfs         /var/lib/k8s        k8sfs
  ├─LV_data-LV_data 55,9G xfs         /data               datafs
  └─LV_data-LV_logs  9,3G xfs         /var/log/containers contlogsfs
sr0                 1024M
```

## Longhorn Storage
Overview off the Longhorn storage setup.
> Not completed yet

### Longhorn Node & Disk Overview
| Node   | IP               | Node tag        | Disk            |  Disk Tag    | Size      | 
| :---:  | :---             | :---            | :---            | :---         | :---:     |
| csrv1  | 192.168.210.95   |  `keycloak`     |                 |              | 82.7 Gi   |
| -->    |                  |                 | `data-disk`     | `keycloak`   | 50.8 Gi   |     
| -->    |                  |                 | `default-disk`  |  -           | 31.9 Gi   |
| wsrv1  | 192.168.23.237   |  `keycloak`     |                 |              | 82.7 Gi   |
| -->    |                  |                 | `data-disk`     | `keycloak`   | 50.8 Gi   |
| -->    |                  |                 | `default-disk`  |  -           | 31.9 Gi   |
| wsrv2  | 192.168.218.100  |  `PostgreSQL`   |                 |              | 82.7 Gi   |
| -->    |                  |                 | `data-disk`     | `PostgreSQL` | 50.8 Gi   |
| -->    |                  |                 | `default-disk`  |  -           | 31.9 Gi   |
| wsrv3  | 192.168.167.156  |  `PostgreSQL`   |                 |              | 82.7 Gi   |
| -->    |                  |                 | `data-disk`     | `PostgreSQL` | 50.8 Gi   |
| -->    |                  |                 | `default-disk`  |  -           | 31.9 Gi   |

> ⚠️ The node IP addresses shown in Longhorn differ from the actual node setup.
> Longhorn retrieves its IPs from the Kubernetes API (`Node.Status.Addresses`), but it does not always select the `Internal-IP`.
> In some cases, it picks the first routable IP based on the network configuration and how `kubelet` was started.
> This behavior cannot be configured directly within Longhorn — there is no setting to explicitly force the IP.
> I plan to address this later by enforcing the correct IPs to ensure consistency across the cluster and documentation.

### Volumes
| Volume Name      | Size | Replicas | Status   | Tag        |
| :---            |:---    |:---      |:---     |  :---           
| `pg-timescale-data` | 8Gi  | 2        | Detached | `PostgreSQL` |
| `keycloak-data`     | 1Gi  | 2        | Released | `keycloak`   |

