# Kubernetes Architecture

Kubernetes (K8s) is an open-source container orchestration platform.

## Control Plane Components

| Component | Role |
|-----------|------|
| **kube-apiserver** | Frontend for the K8s control plane |
| **etcd** | Key-value store for cluster data |
| **kube-scheduler** | Assigns pods to nodes |
| **kube-controller-manager** | Runs controller processes |

## Node Components

| Component | Role |
|-----------|------|
| **kubelet** | Ensures containers are running in a Pod |
| **kube-proxy** | Maintains network rules |
| **Container Runtime** | Runs containers (e.g., containerd) |

## Architecture Overview

```
┌─────────────────────────────────────┐
│         Control Plane               │
│  ┌──────────┐  ┌──────┐            │
│  │API Server│  │ etcd │            │
│  └──────────┘  └──────┘            │
│  ┌──────────────────────────┐      │
│  │ Scheduler + Controllers  │      │
│  └──────────────────────────┘      │
├─────────────────────────────────────┤
│         Worker Nodes                │
│  ┌───────────┐  ┌───────────┐      │
│  │  Node 1   │  │  Node 2   │      │
│  │ kubelet   │  │ kubelet   │      │
│  │ Pod  Pod  │  │ Pod  Pod  │      │
│  └───────────┘  └───────────┘      │
└─────────────────────────────────────┘
```

## Key Concepts

- **Pod** — Smallest deployable unit
- **Service** — Stable network endpoint for Pods
- **Deployment** — Declarative updates for Pods
- **Namespace** — Virtual clusters within a cluster
