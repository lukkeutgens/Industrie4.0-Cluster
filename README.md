# Industry 4.0 Cluster on Debian + Kubernetes
> ⚠️ Work in progress: This repository documents the creation of a personal Industry 4.0 test environment built on four headless Debian 12 virtual machines running via Oracle VirtualBox on a Windows 11 laptop. It serves as a hands-on learning lab to explore open-source technologies. It's an evolving test setup.

After 20+ years as a PLC and SCADA programmer in industrial automation (Siemens, Beckhoff, Omron), I’m now shifting focus. Rather than continuing in pure PLC programming, I’m preparing for roles that bridge Operational Technology (OT) and Information Technology (IT). This project is part of that transition — a way to learn, experiment, and document how modern infrastructure (Linux, Kubernetes, containers) can support industrial environments in a scalable, secure, and maintainable way.

It’s not a production setup — it’s a learning lab. A place to test ideas, break things safely, and understand what works (and what doesn’t).

## Learning Goals of This Project
- Understand open-source ecosystems: Explore what’s available, how it works, and how it can be combined to build resilient industrial platforms.
- Containerize industrial services: Use Kubernetes and Helm to deploy modular, scalable components.
- Avoid vendor lock-in: Favor transparency and self-hosting over proprietary cloud platforms.
- Design for high availability (HA): Build a resilient architecture with redundant storage and smart workload placement.
- Implement security and access control: Apply OS hardening, RBAC, and identity management via Keycloak.
- Capture and visualize industrial data: Use PostgreSQL, TimescaleDB, and Grafana to collect and visualize time-series data.

## Stack Overview
- Debian 12 Headless: Lightweight, stable base OS
- Kubernetes: Orchestration and container runtime (CNI Calico, MetalLB Loadbalancer, Ingress-nginx)
- CoreDNS: DNS and Service Discovery
- Cert-manager: X.509 certificate management for Kubernetes and OpenShift
- Longhorn: Kubernetes service for distributed persistent storage
- PostgreSQL + TimelabsDB: Central & Time-series data storage

## To do
- Keycloak: Identity and access management (and linking it to other services)
- MQTT Broker software? -> Probably EMQX MQTT Broker
- Grafana: Data visualisation
- Prometheus? Is it needed?
- An MQTT-client with OPC-UA or Modbus-TCP on-board?


## Repository Structure
```plaintext
industrie4.0-cluster/
├── 01-common/
│   └── architecture.md              # Cluster overview and component links
├── 02-debian/
│   ├── docs/                        # Debian installation and hardening steps
│   └── files/                       # Custom config files (e.g. interfaces, motd)
├── 03-kubernetes/
│   ├── docs/                        # Kubernetes setup, CNI, Ingress, MetalLB
│   └── files/                       # Helm values, manifests, kubeadm configs
├── 04-longhorn/
|   ├── docs/                        # Longhorn storage setup
│   └── files/                       # Longhorn storage files
├── 05-cert-manager/
|   ├── docs/                        # Cert-manager documentation
│   └── files/                       # Cert-manager files
├── 06-postgresql/    
|   ├── docs/                        # PostgreSQL database setup
│   └── files/                       # Files
├── 07-keycloak/    <- Next to do
│   ├── docs/                        # Keycloak deployment and access control
│   └── files/                       # Helm values, secrets, PVC configs
.
.
.
└── README.md                        # This file
```  
