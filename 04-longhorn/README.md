# Longhorn Persistent Storage for Kubernetes
> ⚠️ Note: When running Kubernetes on VirtualBox, volume provisioning must use the Filesystem access mode instead of Block. This is due to VirtualBox's limitations in handling raw block devices.
> In Longhorn itself, the volume frontend remains a Block device, but Kubernetes mounts it as a filesystem. This ensures compatibility while preserving Longhorn's internal block-level replication.

To ensure reliable and persistent storage across our Industry 4.0 Kubernetes cluster, we deploy Longhorn, a lightweight, distributed block storage system designed specifically for Kubernetes.

Longhorn enables the creation of Persistent Volumes (PVs) that are automatically replicated across multiple nodes. These volumes are then attached to stateful applications, workloads that require data durability, such as databases, monitoring tools, or industrial control services.

By integrating Longhorn, we achieve:
- Data resilience: Application data survives pod crashes, restarts, and rescheduling.
- Node failover: Stateful apps can continue running on other nodes without data loss.
- Cluster robustness: Storage replication enhances fault tolerance and high availability.

This setup is essential for any production-grade Kubernetes environment where data integrity and uptime are critical — especially in industrial automation scenarios where stateful services must remain operational even during infrastructure disruptions.

---

## Installation Overview
In this project, Longhorn will be installed on Kubernetes running on a set of four headless Debian 12 servers inside Oracle VirtualBox. The host is a Windows 11 laptop. 

The installation process consists of two main phases:
1. **Preparation Phase**: Before deploying Longhorn, we need to install additional services and dependencies on each node. These include required kernel modules, system utilities, and network configurations to ensure Longhorn can operate reliably in a virtualized environment.
2. **Preflight Check**: A final validation step is performed to confirm that all nodes meet Longhorn’s prerequisites. This includes verifying disk availability, node labels, system compatibility, and network connectivity.
3. **Longhorn Installation via kubectl**: Longhorn is deployed using a manifest file provided by the Longhorn team. This file is applied via kubectl, which creates the necessary Custom Resource Definitions (CRDs), controllers, and services. Once deployed, Longhorn automatically detects available disks and begins managing storage across the cluster.
4. **Expose Longhorn UI via Ingress**: Longhorn includes a built-in web interface for managing volumes, nodes, backups, and settings. To make this UI accessible externally, we configure an Ingress resource that routes traffic to the Longhorn frontend. This allows secure access via a domain name or IP, often protected with TLS and authentication.
5. **Add Disks (Volumes) to Nodes**: After installation, Longhorn scans each node for available block devices. These can be dedicated virtual disks or partitions mounted under /var/lib/longhorn. Once detected, disks are registered in the Longhorn UI and used to create Persistent Volumes (PVs). These volumes are automatically replicated and attached to stateful workloads.

---

## Folder Contents
> ⚠️ Note: This guide reflects the exact steps I followed during my Kubernetes project. The descriptions may differ slightly from the files/ section, which likely contains the most up-to-date version.

```plaintext
04-longhorn/
├── docs
│   ├── 01-preparation.md            # Preparation Phase steps before installing
│   ├── 02-preflight.md              # Preflight check to see all nodes are ready
│   ├── 03-installation.md           # Actually installing longhorn
│   ├── 04-setup-ui.md               # Setup the graphical user interface
│   └── 05-add-disks.md              # Add the disks (volumes) on the nodes
├── files
│   ├── csrv1-disks.yaml             # Create Ingress Resource
│   ├── longhorn-ingress.yaml        # DNS Resolution
│   ├── resolv.conf                  # Control plane longhorn setup export
│   ├── wsrv1-disks.yaml             # Worker node 1 longhorn setup export
│   ├── wsrv2-disks.yaml             # Worker node 2 longhorn setup export
│   └── wsrv3-disks.yaml             # Worker node 3 longhorn setup export
└── README.md                        # This file
```
  
