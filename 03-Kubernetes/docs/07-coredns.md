# Custom CoreDNS as Domain Server
[CoreDNS Documentation](https://coredns.io)

We’ll extend the default Kubernetes CoreDNS setup by deploying a **custom CoreDNS service** that acts as a domain server.  
This allows us to resolve internal services using DNS names instead of IP addresses — useful for dashboards, monitoring tools, and custom apps.

---

## 1. Create the Deployment File
Create a file named `coredns-custom-depl.yaml` with the following content:
```yaml
kind: ConfigMap
metadata:
  name: coredns-custom-config
  namespace: kube-system
data:
  Corefile: |
    .:53 {
      hosts {
        192.168.50.210 kubedash.iot.keutgens.be
        192.168.50.210 grafana.iot.keutgens.be
        fallthrough
      }
      forward . 8.8.8.8
      log
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns-custom
  namespace: kube-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coredns-custom
  template:
    metadata:
      labels:
        app: coredns-custom
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - coredns-custom
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: coredns
        image: coredns/coredns:latest
        args: [ "-conf", "/etc/coredns/Corefile" ]
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
      volumes:
      - name: config-volume
        configMap:
          name: coredns-custom-config
---
apiVersion: v1
kind: Service
metadata:
  name: coredns-custom
  namespace: kube-system
  annotations:
    metallb.universe.tf/address-pool: lb-pool
spec:
  type: LoadBalancer
  selector:
    app: coredns-custom
  ports:
  - name: dns-udp
    protocol: UDP
    port: 53
    targetPort: 53
  - name: dns-tcp
    protocol: TCP
    port: 53
    targetPort: 53
```
This configuration creates a custom CoreDNS deployment with static host mappings and forwards unknown queries to Google DNS (8.8.8.8). 
It uses MetalLB to expose the DNS service via a LoadBalancer IP.

---

## 2. Apply the Deployment
```bash
kubectl apply -f coredns-custom-depl.yaml
```

---

## 3. Verify the Service
Check that the service is running and has received an IP from MetalLB:
```bash
kubectl get svc -n kube-system coredns-custom
```
Expected output:
```bash
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
coredns-custom   LoadBalancer   10.x.x.x       192.168.50.211  53:53/UDP,TCP  XXm
```






