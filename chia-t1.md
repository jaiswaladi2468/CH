Kubernetes manifests for running a Docker registry with the specified requirements:

1. **PersistentVolumeClaim (pv-claim.yaml)**:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: registry-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

2. **Deployment (registry-deployment.yaml)**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: registry
  template:
    metadata:
      labels:
        app: registry
    spec:
      containers:
        - name: registry
          image: registry:2
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: registry-storage
              mountPath: /var/lib/registry
        - name: redis
          image: redis:latest
          ports:
            - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: registry-service
spec:
  selector:
    app: registry
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: ClusterIP
```

3. **Secret (registry-secret.yaml)**:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: Opaque
data:
  REGISTRY_AUTH: base64_encoded_auth
```

4. **Garbage Collection CronJob (gc-cronjob.yaml)**:

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: registry-gc
spec:
  schedule: "0 1 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: registry-gc
            image: registry:2
            args:
            - garbage-collect
            - /etc/docker/registry/config.yml
          restartPolicy: OnFailure
```

5. **Ingress (registry-ingress.yaml)**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: registry-ingress
spec:
  rules:
  - host: registry.example.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: registry-service
            port:
              number: 80
```

6. **Combined with Volume and Redis (registry-all.yaml)**:

```yaml
# Combine the above manifests and add PersistentVolumeClaim and Redis configurations
```

To apply these manifests, you can use the following commands:

```bash
kubectl apply -f pv-claim.yaml
kubectl apply -f registry-all.yaml
```

Please note the following:

- Make sure you replace `base64_encoded_auth` in the `registry-secret.yaml` with actual base64 encoded credentials if required.
- Ensure you have Redis service running or adjust the Redis configuration as needed.
- Replace `registry.example.com` with your actual domain in the Ingress manifest.
- Make sure you have the Docker images (`registry:2` and `redis:latest`) available in your container registry.
- Ensure you have sufficient privileges to create these resources in your Kubernetes cluster.

Let me know if you encounter any issues!
