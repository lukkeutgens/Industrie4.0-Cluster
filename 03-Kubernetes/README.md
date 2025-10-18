# Kubernetes
Official Site: https://kubernetes.io/
Tutorial: https://medium.com/@sirtcp/deploying-a-three-node-kubernetes-cluster-on-debian-12-with-kubeadm-and-rook-ceph-for-persistent-eb080f31d3fc

Kubernetes, also known as K8s, is an open-source system for automating the deployment, scaling, and management of containerized applications.

---

## Why Kubernetes?
- Fully open-source with a large and active community
- Can run on-premises and is supported by most cloud providers
- Supports High Availability (HA) clusters with redundant data options
- Highly flexible in scaling the cluster up or down
- Self-healing capabilities when containers fail
- Supports rolling updates for zero-downtime deployments

## Why Not Kubernetes
- Complex to set up and maintain (see installation notes below)
- Requires additional services such as a CNI plugin, load balancer, ingress controller, and persistent storage
- Steep learning curve
- Resource overhead — consumes CPU and memory to maintain cluster operations
- Security configuration must be handled carefully to avoid vulnerabilities
- May be overkill for small-scale or single-node environments

## Installation
For this project, I will set up a Kubernetes cluster on four virtual Debian 12 headless servers running in Oracle VirtualBox on my Windows 11 laptop.

The installation process is divided into three chapters:
- Pre-installation: Preparing the servers and nodes
- Installation: Installing Kubernetes components on all nodes
- Post-installation: Installing additional packages to create a fully functional cluster

---

## Folder Contents
> ⚠️ Note: This guide reflects the exact steps I followed during my Kubernetes project. The descriptions may differ slightly from the files/ section, which likely contains the most up-to-date version.

```plaintext
03-kubernetes/
├── docs
│   ├── 01-pre-installation.md                   # Pre-installation steps before installing
│   ├── 02-install-kubernetes.md                 # Installation off kubernetes itself
│   ├── 03-install-cni-calico.md                 # A CNI-Plugin is needed within Kubernetes
│   ├── 04-post-installation.md                  # Post installation steps setting up kubernetes
│   ├── 05-metallb-loadbalancer.md               # Installing loadbalancer plugin
│   ├── 06-nginx-ingress-controller.md           # Installing ingress-controller
│   ├── 07-coredns.md                            # Setup DNS controller
│   └── 08-Kubernetes-dashboard.md               # Install the Kubernetes Dashboard
├── files
│   ├── calico
│   │   ├── 01-operator-crds.yaml                # Calico CRDs for operator
│   │   ├── 02-tigera-operator.yaml              # Tigera operator deployment
│   │   ├── 03-custom-resources.yaml             # Calico custom resources
│   │   ├── ippool.yaml                          # Calico IP pool definition
│   │   └── operator-tigera-installation.yaml    # Tigera installation manifest
│   ├── config.toml                              # Containerd config file
│   ├── coredns-custom-depl.yaml                 # Custom CoreDNS deployment
│   ├── fstab                                    # Mount points config
│   ├── ingress-nginx-values.yaml                # Helm values for ingress-nginx
│   ├── kubeconfig-csrv1                         # Kubeconfig for admin user
│   ├── kubernetes-dashboard
│   │   ├── admin-user.yaml                      # Dashboard admin user
│   │   ├── dashboard-ingress.yaml               # Ingress for dashboard access
│   │   └── dashboard-web-service.yaml           # Web service for dashboard
│   ├── metallb
│   │   ├── metallb-advertisement.yaml           # Layer 2 advertisement config
│   │   └── metallb-ip-pool.yaml                 # IP address pool for MetalLB
│   ├── metrics-server.yaml                      # Metrics-server manifest
│   ├── sysctl.conf                              # Kernel tuning config
│   └── ufw                                      # UFW default config
└── README.md                                    # This file

```


