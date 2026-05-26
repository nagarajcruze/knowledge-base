# Kubernetes Notes

My personal notes, architecture diagrams, object manifests, and command cheat sheets for Kubernetes.

---

## 1. Kubernetes Architecture

Kubernetes (K8s) is an open-source container orchestration platform. It is structured around a Control Plane that manages worker nodes running the actual containerized applications.

```text
┌────────────────────────────────────────────────────────┐
│                     Control Plane                      │
│   ┌──────────────┐   ┌──────────────┐   ┌──────────┐   │
│   │ API Server   │   │  Controller  │   │Scheduler │   │
│   └──────────────┘   └──────────────┘   └──────────┘   │
│   ┌──────────────┐                                     │
│   │     etcd     │                                     │
│   └──────────────┘                                     │
├────────────────────────────────────────────────────────┤
│                     Worker Nodes                       │
│   ┌────────────────────────┐  ┌────────────────────┐   │
│   │         Node 1         │  │       Node 2       │   │
│   │  ┌───────┐  ┌───────┐  │  │  ┌───────┐         │   │
│   │  │kubelet│  │ proxy │  │  │  │kubelet│  ...    │   │
│   │  └───────┘  └───────┘  │  │  └───────┘         │   │
│   │  ┌───────┐  ┌───────┐  │  │  ┌───────┐         │   │
│   │  │ Pod   │  │ Pod   │  │  │  │ Pod   │         │   │
│   │  └───────┘  └───────┘  │  │  └───────┘         │   │
│   └────────────────────────┘  └────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

### Control Plane Components
- **`kube-apiserver`**: The frontend gateway for the control plane. It exposes the Kubernetes API and handles external and internal requests.
- **`etcd`**: A highly available, distributed key-value store holding all cluster metadata and configuration details (the single source of truth).
- **`kube-scheduler`**: Assigns newly created pods to worker nodes based on resource availability, node affinity, and policy constraints.
- **`kube-controller-manager`**: Runs control loops to regulate cluster state (e.g., node health, replication counts, namespace configurations).

### Node Components
- **`kubelet`**: An agent running on each worker node ensuring that containers described in PodSpecs are running and healthy.
- **`kube-proxy`**: Manages network rules on nodes, enabling network communication to pods from inside or outside the cluster.
- **Container Runtime**: The underlying engine that executes containers (e.g., `containerd`).

---

## 2. Core Kubernetes Objects Map

For beginners, here is a quick overview of the key objects that define state in a Kubernetes cluster:

- **Pod**: The smallest, most fundamental deployable unit, wrapping one or more tightly coupled containers.
- **ReplicaSet**: A low-level controller ensuring a specified number of identical Pod replicas are running at any given time. (Typically managed by Deployments).
- **Deployment**: Provides declarative updates for Pods and ReplicaSets, managing stateless application lifecycles.
- **Service**: Exposes a set of Pods as a network service using a stable IP address and load balancer.
- **StatefulSet**: Used for managing stateful applications (like databases) that require stable, unique network identifiers and persistent, sticky storage.
- **DaemonSet**: Ensures that all (or a subset of) nodes run a single copy of a specific Pod (e.g., logging/monitoring agents).
- **ConfigMap & Secret**: Externalize configuration data (ConfigMaps for non-sensitive, Secrets for sensitive credentials) to keep container images stateless.
- **Jobs & CronJobs**: Run finite tasks to completion (Jobs run once, CronJobs run on a scheduled cron-like time).

---

## 3. Pods

Pods are the primary building blocks of Kubernetes. They encapsulate container startup behaviors, resource allocations, and shared networks.

### Example Pod Manifest (`pod.yaml`)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
  labels:
    app: my-app
    env: prod
spec:
  containers:
    - name: app-container
      image: coolgourav147/nginx-custom
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

### Pod Management Commands
Apply or create a pod:
```bash
kubectl apply -f pod.yaml
# or
kubectl create -f pod.yaml
```

Generate a baseline Pod template YAML file imperatively:
```bash
kubectl run myfirstpod --image=testdockerimage --dry-run=client -o yaml > myfirstyaml.yaml
```

Access the interactive terminal of a running container:
```bash
kubectl exec -it test-vol -- /bin/bash
```

Run diagnostics inside a container:
```bash
kubectl exec test-vol -- env
kubectl exec test-vol -- ps aux
kubectl exec test-vol -- ls /
```

Force delete a pod immediately (without waiting for graceful termination):
```bash
kubectl delete pod <pod-name> --grace-period=0 --force
```

---

## 4. Deployments

Deployments manage stateless application rollouts, scaling, and update strategies.

### Example Deployment Manifest (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.16
          ports:
            - containerPort: 8080
```

### Deployment Strategies
Kubernetes supports several strategies for deploying and updating applications:

- **Recreate**:
  - **Process**: All existing Pods are terminated first, then the new version's Pods are created.
  - **Use Case**: When databases or application state cannot run two different versions concurrently.
  - **Downside**: Causes downtime equal to the time it takes for old Pods to shut down and new Pods to start.
- **Rolling Update (Default)**:
  - **Process**: Gradually replaces instances of the old Pod version with the new version.
  - **Use Case**: Standard microservice updates requiring zero downtime.
  - **Control parameters**:
    - `maxUnavailable`: Maximum number of Pods that can be unavailable during the update.
    - `maxSurge`: Maximum number of Pods that can be created above the desired replica count.
- **Blue-Green Deployment (Requires external routing)**:
  - **Process**: Deploys the new version (Green) alongside the old running version (Blue) in complete isolation. Once the Green deployment is fully tested, traffic is switched instantly at the router/service layer.
  - **Upside**: Near-zero downtime and instant rollback capabilities.
  - **Downside**: Requires double the server resource footprint during the transition.
- **Canary Deployment (Requires progressive traffic management)**:
  - **Process**: Deploys the new version to a tiny portion of the cluster (e.g., 5-10% of users). Once the performance is validated (no errors, low latency), the rollout is gradually expanded to all hosts.

### Rollout Management Commands
Scale a deployment:
```bash
kubectl scale --replicas=4 deployment/nginx-deployment
```

Annotate deployments to track changes in the rollout history:
```bash
kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="Updating image version to 1.16"
```

View rollout revisions:
```bash
kubectl rollout history deployment/nginx-deployment
```

Roll back a deployment to a specific revision:
```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

Check current rollout status:
```bash
kubectl rollout status deployment/nginx-deployment
```

---

## 5. Application High Availability (HA) using Pods

In Kubernetes, Application High Availability ensures that your application remains responsive and online even if individual containers crash, processes freeze, or worker nodes go offline. This is managed using Pod-level configurations:

1. **Replication & Scaling**:
   - Running multiple copies of your application (setting `replicas` $\ge 2$ in a Deployment) ensures that if one Pod fails, others are still active to handle traffic.
2. **Self-Healing (ReplicaSet Controller)**:
   - If a Pod crashes or its node dies, the ReplicaSet controller detects the mismatch between the desired state (e.g., 3 replicas) and actual state (2 replicas) and automatically spins up a new Pod on a healthy node.
3. **Health Probes (Container Health Checks)**:
   - **Liveness Probe**: Monitors if the containerized process is still running. If a Liveness probe fails, Kubernetes automatically kills and restarts the Pod container.
   - **Readiness Probe**: Monitors if the application is fully ready to accept user requests (e.g., after loading databases or warming up caches). If this probe fails, the Pod is temporarily removed from the Service load balancer endpoints so users do not receive error screens.
   - **Startup Probe**: Disables liveness and readiness checks during slow application startup periods to prevent premature restarts.
4. **Pod Anti-Affinity**:
   - Ensures that replicas of the same application are not scheduled onto the same physical worker node. This prevents a single hardware crash from bringing down all instances of your service.
5. **Pod Disruption Budgets (PDB)**:
   - Specifies the minimum number of Pods that must remain healthy during voluntary disruptions (such as node drain operations during cluster maintenance/upgrades).

---

## 6. Services

Services expose a set of Pods as a network service. Since Pods are ephemeral and their IPs can change, a Service provides a single, stable network endpoint and load balances traffic across matching Pods.

### Service Types

| Type | Description |
| :--- | :--- |
| **`ClusterIP`** | Exposes the service on an internal-only IP (default). Accessible only within the cluster. |
| **`NodePort`** | Exposes the service on each static port of each Node's IP. Accessible externally. |
| **`LoadBalancer`** | Exposes the service externally using a cloud provider's network load balancer. |
| **`ExternalName`** | Maps the service to a DNS name (using a `CNAME` record). |

### Example Service Manifest (`service.yaml`)
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

### Service Management Commands
Expose a deployment using NodePort:
```bash
kubectl expose deployment web --type=NodePort --port=8080
```

Open a Minikube service in your default browser:
```bash
minikube service mongo-express-service
```

Forward local ports to a service:
```bash
kubectl port-forward svc/mongo-svc 32000:27017
```

SSH into the Minikube virtual machine:
```bash
minikube ssh
```

---

## 7. Namespaces

Namespaces divide cluster resources among multiple users or teams (virtual clusters inside the same physical cluster).

### Namespace Commands
List all resources across all namespaces:
```bash
kubectl get all --all-namespaces
```

Set the default namespace for subsequent CLI commands:
```bash
kubectl config set-context --current --namespace=nginx
```

Access services in different namespaces using DNS resolution:
```bash
curl service_name.namespace_name:port/path/to/api
```

### Example Namespace Manifest
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
```

---

## 8. Secrets & Ingress

### Secrets
Secrets let you store and manage sensitive information (such as passwords, OAuth tokens, and ssh keys) securely.

Generate a Secret template YAML file:
```bash
kubectl create secret generic database --from-literal=DB_PASSWORD=hello@123 -n default --dry-run=client -o yaml > secret.yaml
```

Example Secret manifest:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: grafana-pg-user
type: kubernetes.io/basic-auth
data:
  username: {{ .Values.postgres.user | b64enc }}
  password: {{ .Values.postgres.password | b64enc }}
```

### Ingress
Ingress manages external HTTP/HTTPS traffic to internal services, supporting host and path-based routing.

Example Ingress manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-service
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:  
              service:
                name: adminer
                port:
                  number: 8080
```

Test Ingress routing via curl:
```bash
curl --resolve "hello-world.example:80:$(minikube ip)" -i http://hello-world.example
```

---

## 9. Storage: Persistent Volumes & Node Affinity

### Persistent Volumes (PV) and Claims (PVC)
PVs are cluster-level storage resources provisioned by administrators. PVCs are user requests for those storage resources. They maintain a strict **1:1 relationship**.

### Local Volumes with Node Affinity
If using `hostPath` for testing, you can bypass PVs and PVCs by specifying `nodeAffinity` (pinning the pod to a host node), `volumeMounts`, and `volumes` directly inside the Deployment YAML:
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - ubuntu18-kubeadm-worker1
  containers:
    - name: nginx
      image: nginx:latest
      volumeMounts:
        - mountPath: /home # Mount path inside container
          name: vol-postgres
  dnsPolicy: ClusterFirstWithHostNet
  hostNetwork: true
  volumes:
    - name: vol-postgres
      hostPath:
        path: /tmp/postgres # Path on the host node
```

*Note: You can give higher priority to pods using local volumes by specifying `spec.priorityClassName`.*

---

## 10. Diagnostics & Events

Sort cluster events by creation time:
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Fetch a pod's unique ID (UID):
```bash
kubectl get po pod_name -o jsonpath='{.metadata.uid}'
```

Locate host-level container paths on the worker node:
```bash
ls /var/lib/kubelet/pods/{uid}
```

*Note: Self-signed certificates are typically stored at `/usr/local/share/ca-certificates/ca.crt` on nodes.*
