# Deploying Hiphops to Kubernetes

> Pardon our mess. This area of the docs is still under construction

Deploying the hops executable to Kubernetes is straightforward.

You will need to deploy a container that contains the hops executable which is available at Docker Hub under `hiphops/hiphops` and along side this your key file and your `.hops` file(s).

This is a basic sample deployment file:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hops
  template:
    metadata:
      labels:
        app: hops
    spec:
      containers:
      - name: hops
        # Hops image
        image: hiphops/hiphops:v0.2.0
        command: ["/hops"]
        args: ["start", "--address=0.0.0.0:8916", "--keyfile=/root/hiphops-key/hiphops.key", "--hops=/root/hops-conf/"]
        volumeMounts:
        - name: hops-conf
          mountPath: /root/hops-conf/
        - name: hiphops-key
          mountPath: /root/hiphops-key
        ports:
        - containerPort: 8916
        startupProbe:
          httpGet:
            path: /health
            port: 8916
          periodSeconds: 1
          failureThreshold: 30
        livenessProbe:
          httpGet:
            path: /health
            port: 8916
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 15
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /health
            port: 8916
            scheme: HTTP
          initialDelaySeconds: 5
          timeoutSeconds: 1
      volumes:
      - name: hops-conf
        configMap:
          name: hops-conf
          items:
            - key: main.hops
              path: main.hops
      - name: hiphops-key
        secret:
          secretName: hiphops-key
```

You will have to define the volumes for the hops configuration and the key file.
