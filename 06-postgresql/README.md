# PostgreSQL + TimescaleDB
PostgreSQL is a powerful open-source relational database known for its reliability, extensibility, and standards compliance. It supports advanced features like JSON fields, full-text search, and custom extensions.
TimescaleDB is a PostgreSQL extension designed for efficient storage and querying of time-series data. It adds automatic partitioning, compression, and performance optimizations for metrics, logs, and sensor data.
This module deploys a high-availability PostgreSQL database with TimescaleDB extension for time-series workloads, tailored for industrial metrics and future integration with Keycloak and MQTT brokers.
- [PostgreSQL official site](https://www.postgresql.org/)
- [TimescaleDB GitHub repository](https://github.com/timescale/timescaledb)

---

## Why PostgreSQL?
PostgreSQL was selected as the core database engine for the following reasons:
- **Relational + JSON support**: Combines structured schema with flexible JSON fields for hybrid data models.
- **Keycloak compatibility**: Supports integration with identity providers via JDBC and standard auth flows.
- **Open-source & widely supported**: Mature ecosystem, strong community, and long-term viability.
- **TimescaleDB extension**: Adds native time-series capabilities, ideal for metrics from PLCs and industrial sensors.
- **Strong replication & failover options**: Supports synchronous and asynchronous replication, WAL shipping, and HA setups.

---

## Deployment Strategy
We deploy PostgreSQL + TimescaleDB as a **StatefulSet** with persistent volumes managed by Longhorn. This ensures:
- **Stable pod identity**: Each replica maintains a consistent network name and volume binding.
- **Persistent storage**: Longhorn volumes survive pod restarts and node failures.
- **High availability**: Future support for read replicas and automated failover.
- **Observability**: Metrics can be exposed via Prometheus exporters and visualized in Grafana.

> Helm is deliberately avoided to maintain full control over manifests and support gradual learning of Kubernetes primitives.

---

## ⚠️ Note on HA and Patroni
Patroni was deliberately skipped in this setup. At this stage, my own Kubernetes knowledge isn’t yet mature enough to confidently deploy and manage a full HA PostgreSQL cluster with distributed consensus. Additionally, VirtualBox may impose limitations that complicate reliable failover and network coordination between nodes. For now, a single TimescaleDB instance with Longhorn-backed storage and Kubernetes-managed pod recovery offers a simpler foundation to build on.

---

## Folder Contents
```text
├── docs
│   ├── 01-create-volume.md                # Create a Longhorn volume, then register it in Kubernetes
│   ├── 02-install-postgre.md              # Deploy PostgreSQL + TimescaleDB using Secret, Service, and StatefulSet
│   └── 03-postgre-help.md                 # Quick reference for accessing and working with PostgreSQL
├── files
│   ├── pg-timescale-pvc.yaml              # PersistentVolumeClaim bound to created Longhorn volume
│   ├── pg-timescale-pv.yaml               # PersistentVolume definition for Longhorn volume
│   ├── timescaledb-deploy.yaml            # StatefulSet for PostgreSQL + TimescaleDB container
│   ├── timescaledb-secret.yaml            # Kubernetes Secret with initial database credentials
│   └── timescaledb-service.yaml           # Headless Service for stable DNS and pod discovery
└── README.md                              # This file
```
