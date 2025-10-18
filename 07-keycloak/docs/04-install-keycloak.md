# Keycloak Deployment
In previous steps we already created the volume in Longhorn, connected it to Kubernetes.
Created a new keycloak database in PostgreSQL and create a Kubernetes secret to login into this database and also with the keycloak admin.

## Add the keycloak domain
First we will change the CoreDNS file to add the keycloak domain. This file can be found under `03-Kubernetes/files`.
```bash
vi coredns-custom-depl.yaml
```
Add the line `192.168.50.210 keycloak.iot.keutgens.be` like below.
```yaml
data:
  Corefile: |
    .:53 {
      hosts {
        192.168.50.210 kubedash.iot.keutgens.be
        192.168.50.210 grafana.iot.keutgens.be
        192.168.50.210 longhorn.iot.keutgens.be
        192.168.50.210 keycloak.iot.keutgens.be
        fallthrough
      }
      forward . 8.8.8.8
      log
    }
---
```


