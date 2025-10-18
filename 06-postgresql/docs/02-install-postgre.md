# PostgreSQL + TimescaleDB Installation
This guide describes how to deploy PostgreSQL with TimescaleDB in your Kubernetes cluster using:
- A Kubernetes Secret for credentials
- A headless Service for internal DNS
- A StatefulSet for persistent deployment
- A manually bound PVC linked to a Longhorn volume (`pg-timescale-pvc`)

---

## 1. Create Secret
Create the file `timescaledb-secret.yaml`:
```bash
vi timescaledb-secret.yaml
```
With content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: timescaledb-secret
  namespace: postgresql
type: Opaque
stringData:
  POSTGRES_USER: iotadmin
  POSTGRES_PASSWORD: ******        # Replace with your password
  POSTGRES_DB: postgres
```
Apply the secret:
```bash
kubectl apply -f timescaledb-secret.yaml
```
This secret will be injected into the container as environment variables, allowing PostgreSQL to initialize with the correct user and database.

---

## 2. Create Headless Service
Create the file `timescaledb-service.yaml`:
```bash
vi timescaledb-service.yaml
```
With content:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: timescaledb
  namespace: postgresql
spec:
  ports:
    - port: 5432
      targetPort: 5432
  selector:
    app: timescaledb
  clusterIP: None
```
Apply the service:
```bash
kubectl apply -f timescaledb-service.yaml
```
This headless service enables stable DNS resolution for the pod (timescaledb-0.postgresql.svc.cluster.local) and is required by the StatefulSet.

---

## 3. Deploy StatefulSet
Create the file `timescaledb-deploy.yaml`:
With content:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: timescaledb
  namespace: postgresql
spec:
  serviceName: timescaledb
  replicas: 1
  selector:
    matchLabels:
      app: timescaledb
  template:
    metadata:
      labels:
        app: timescaledb
    spec:
      containers:
        - name: timescaledb
          image: timescale/timescaledb:latest-pg17
          ports:
            - containerPort: 5432
          env:
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          envFrom:
            - secretRef:
                name: timescaledb-secret
          volumeMounts:
            - name: pgdata
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: pgdata
          persistentVolumeClaim:
            claimName: pg-timescale-pvc
```
Apply the StatefulSet:
```bash
kubectl apply -f timescaledb-deploy.yaml
```
This will deploy a single TimescaleDB pod (timescaledb-0) and mount the PVC pg-timescale-pvc, which is bound to the manually created Longhorn volume postgre-data.

--- 

## 4. Check PostgreSQL is up and running
After applying the manifests, verify that the pod, PVC, and volume are correctly initialized and bound.
### 4.1 Check pod status:
```bash
kubectl get pod -n postgresql -o wide
```
You should see:
```bash
NAME            READY   STATUS    RESTARTS   AGE   IP                NODE
timescaledb-0   1/1     Running   0          ...   192.168.x.x       wsrvX
```
If the pod is stuck in Pending, continue with the checks below.

### 4.2 Check PVC status
```bash
kubectl get pvc pg-timescale-pvc -n postgresql
```
Expected output:
```bash
NAME               STATUS   VOLUME         CAPACITY   ACCESS MODES 
pg-timescale-pvc   Bound    postgre-data   8Gi        RWO         
```

### 4.3 Check PV status
If the PVC is Pending, check if the corresponding PV exists and is available:
```bash
kubectl get pv | grep postgre-data
```
Check PV details:
```
kubectl describe pv postgre-data
```
Look for:
- Status: Bound
- Claim: postgresql/pg-timescale-pvc
- CSI VolumeHandle: postgre-data

### 4.4 Check pod events and scheduling issues
```bash
kubectl describe pod timescaledb-0 -n postgresql
```
Look for warning events such as:
- FailedScheduling: pod has unbound immediate PersistentVolumeClaims
- VolumeBinding: PVC not found
- MountVolume.SetUp failed

These indicate that the PVC is not yet bound, or the volume is not attached.

### 4.5 Check Longhorn UI
In the Longhorn dashboard:
Volume postgre-data should be:
- Attached to the node where the pod is running
- Healthy
- Replica count: 2
- Disk tag: PostgreSQL

If the volume is still Detached, Kubernetes may not have triggered the CSI attach. Restart the pod to retry:
```bash
kubectl delete pod timescaledb-0 -n postgresql --grace-period=0 --force
```

--- 

## 5. Result
After deployment:
- The pod timescaledb-0 should be in Running state
- The PVC should be Bound to postgre-data
- The Longhorn volume should be Attached and Healthy





