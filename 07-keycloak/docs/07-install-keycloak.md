# Keycloak Deployment Configuration
This deployment launches the Keycloak container inside the `keycloak` namespace, with persistent storage, database connectivity, and health probes. It is designed for production-grade use with Ingress and TLS.

---

## ⚠️ Image Compatibility Notice
We are using an older Keycloak image version: `quay.io/keycloak/keycloak:22.0.5`

This version was chosen because it supports the --auto-build startup option, which is no longer available in newer Keycloak releases. The --auto-build flag allows Keycloak to automatically build its configuration from environment variables, simplifying container setup and avoiding manual configuration files.

Future upgrades may require switching to a custom build or using the build command explicitly. Always verify compatibility before updating the image tag.

> I have tried the newer images first with first deploying a build container and then the normal container but for still an unknown reason it never fully got it working. During debugging, I lowered the image version and tried the "auto-build" and got it working. For me, this is fine now because this is just for testing.

---

## Create deployment
Create file:
```bash
vi keycloak-deploy.yaml
```
Add content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloak
  namespace: keycloak
  labels:
    app: keycloak
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloak
  template:
    metadata:
      labels:
        app: keycloak
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
      initContainers:
        - name: init-pvc-layout
          image: busybox:1.36
          command: ["sh", "-c"]
          args:
            - >
              mkdir -p /mnt/conf /mnt/providers /mnt/themes;
              chown -R 1000:1000 /mnt 2>/dev/null || true;
              echo "init done";
          volumeMounts:
            - name: keycloak-data
              mountPath: /mnt
      containers:
        - name: keycloak
          # image die --auto-build ondersteunt
          image: quay.io/keycloak/keycloak:22.0.5
          command: ["/opt/keycloak/bin/kc.sh"]
          args: ["start", "--auto-build"]
          ports:
            - name: http
              containerPort: 8080
          envFrom:
            - secretRef:
                name: keycloak-secret
          env:
            - name: KC_DB
              value: postgres
            - name: KC_DB_URL_HOST
              value: timescaledb.postgresql.svc.cluster.local
            - name: KC_DB_URL_DATABASE
              value: keycloak
            - name: KC_DB_URL_PORT
              value: "5432"
            - name: KC_PROXY
              value: edge
            - name: KC_PROXY_ADDRESS_FORWARDING
              value: "true"
            - name: KC_HTTP_ENABLED
              value: "true"
            - name: KC_HTTP_RELATIVE_PATH
              value: /
            - name: KC_HOSTNAME
              value: keycloak.iot.keutgens.be
            - name: KC_HOSTNAME_STRICT
              value: "false"
            - name: KC_HEALTH_ENABLED
              value: "true"
            - name: KC_METRICS_ENABLED
              value: "true"
          volumeMounts:
            - name: keycloak-data
              mountPath: /opt/keycloak/conf
              subPath: conf
            - name: keycloak-data
              mountPath: /opt/keycloak/providers
              subPath: providers
            - name: keycloak-data
              mountPath: /opt/keycloak/themes
              subPath: themes
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
              httpHeaders:
                - name: Host
                  value: keycloak.iot.keutgens.be
            initialDelaySeconds: 300
            periodSeconds: 20
            timeoutSeconds: 5
            failureThreshold: 6
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
              httpHeaders:
                - name: Host
                  value: keycloak.iot.keutgens.be
            initialDelaySeconds: 600
            periodSeconds: 30
            timeoutSeconds: 5
            failureThreshold: 6
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "1000m"
              memory: "2Gi"
      volumes:
        - name: keycloak-data
          persistentVolumeClaim:
            claimName: keycloak-pvc
```
Apply the deployment:
```bash
kubectl apply -f keycloak-deploy.yaml
```



