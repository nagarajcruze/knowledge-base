# Pods & Services

## Pods

A Pod is the **smallest deployable unit** in Kubernetes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  containers:
    - name: app
      image: my-app:1.0
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "128Mi"
          cpu: "250m"
        limits:
          memory: "256Mi"
          cpu: "500m"
```

## Services

| Type | Description |
|------|-------------|
| **ClusterIP** | Internal-only access (default) |
| **NodePort** | Exposes on each node's IP |
| **LoadBalancer** | Cloud load balancer |
| **ExternalName** | Maps to a DNS name |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```
